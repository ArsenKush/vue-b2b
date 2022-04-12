<template>
  <div class="ui-item-tile">
    <div class="ui-item-view">
      <v-btn
        :color="itemQuantity(item.itemNo) > 0 ? 'orange' : 'grey'"
        class="favourite-icon"
        flat
        icon
        @click.prevent="addToFavourite(item)"
      >
        <v-icon v-if="itemQuantity(item.itemNo) > 0">
          mdi-bookmark
        </v-icon>
        <v-icon v-else>
          mdi-bookmark-outline
        </v-icon>
      </v-btn>
      <template v-if="!garageMode">
        <garage-icon :item-no="item.itemNo" />
      </template>
      <!-- <offer-icon
        v-if="sale || action || (item.inAction && item.inAction.length)"
        :offers="item.inAction"
        :sale="sale"
        :action="action"
      /> -->
      <div class="tile-spec-icons-block">
        <offer-icons
          v-if="
            sale ||
              action ||
              (item.inAction && item.inAction.length) ||
              specprice ||
              discount
          "
          :offers="item.inAction"
          :disc-group="item.discGroup"
          :specprice="specprice"
          vertical
        />
      </div>

      <!-- <v-btn
        color="red lighten-2"
        flat
        icon
        style="position:absolute; left:-8px; top:24px; z-index:1;"
      >
        <v-icon>mdi-brightness-percent</v-icon>
      </v-btn>-->

      <router-link :to="url(item)">
        <v-img :transition="false" contain :src="imgSrc(item)" />
        <div class="item-data">
          <b>{{ item.itemNo }}</b>
          <small>- {{ item.brand }}</small>
          <small>
            <br />
            {{ item.description }}
          </small>
        </div>
      </router-link>

      <div class="item-data-price">
        <template v-if="!userInRole('HideRetail')">
          <v-tooltip
            v-model="showPrice"
            top
            open-delay="2000"
            close-delay="1"
            open-on-hover="false"
            z-index="16"
          >
            <span
              slot="activator"
              :class="['price ', { spec: !!specPriceItems[item.itemNo] }]"
              @click.stop="showPrice = !showPrice"
            >
              {{
                parseInt(
                  item.increaseFactor == 0
                    ? item.retail
                    : item.price * item.increaseFactor
                )
              }}
              <sup>
                {{
                  getPriceDecimal(
                    item.increaseFactor == 0
                      ? item.retail
                      : item.price * item.increaseFactor
                  )
                }}
              </sup>
            </span>
            <span>
              <span v-if="item.increaseFactor != 0">
                {{ $t('Price Increase Factor') }}:
                {{ item.increaseFactor.toFixed(2) }}
                <br />
              </span>
              {{ $t('Retail:') }}
              {{
                item.increaseFactor == 0
                  ? item.retail
                  : item.price * item.increaseFactor
              }}
              <br />
              {{ $t('Purchase price:') }}
              <span class="title orange--text">{{
                item.price.toFixed(2)
              }}</span>
            </span>

            <template v-if="specPriceItems[item.itemNo]">
              <hr size="1" />
              <span class="spec-price">
                {{ $t('Special price') }}
              </span>
              <br />
              <!-- {{$t('Price for {1} ',{})}} -->
              <span v-for="(sp, j) in specPriceItems[item.itemNo]" :key="j">
                {{
                  $t('from {qty} {uom}: {price}', {
                    qty: sp.minimumQuantity,
                    uom: item.salesUoM,
                    price: (item.retail * (1 - sp.lineDiscount)).toFixed(2)
                  })
                }}
                <br />
              </span>
            </template>
          </v-tooltip>
        </template>

        <template v-else>
          <v-tooltip
            v-model="showPrice"
            top
            open-delay="2000"
            close-delay="1"
            open-on-hover="false"
            z-index="16"
          >
            <span
              slot="activator"
              :class="['price ', { spec: !!specPriceItems[item.itemNo] }]"
              @click.stop="showPrice = !showPrice"
            >
              {{ parseInt(item.price) }}
              <sup>
                {{ getPriceDecimal(item.price) }}
              </sup>
            </span>
            <span>
              {{ $t('Price:') }}
              {{ item.price }}
            </span>

            <template v-if="specPriceItems[item.itemNo]">
              <hr size="1" />
              <span class="spec-price">
                {{ $t('Special price') }}
              </span>
              <br />
              <!-- {{$t('Price for {1} ',{})}} -->
              <span v-for="(sp, j) in specPriceItems[item.itemNo]" :key="j">
                {{
                  $t('from {qty} {uom}: {price}', {
                    qty: sp.minimumQuantity,
                    uom: item.salesUoM,
                    price: (item.retail * (1 - sp.lineDiscount)).toFixed(2)
                  })
                }}
                <br />
              </span>
            </template>
          </v-tooltip>
        </template>

        <v-tooltip
          v-model="show"
          left
          open-delay="100"
          close-delay="1"
          open-on-hover="false"
          z-index="16"
        >
          <div slot="activator" @click.stop="show = !show">
            <!-- st.Q.startsWith('>') ? 'green2' : st.Q != 0 ? 'green' : '' -->
            <template v-if="item.stock">
              <div
                v-for="st in parseStock(item)"
                :key="st.L"
                :class="[
                  'cube',
                  st.Q.startsWith('>')
                    ? 'green2'
                    : st.Q != 0
                    ? 'green'
                    : st.R != 0
                    ? 'orange'
                    : ''
                ]"
              >
                {{ st.Q.substr(0, 1).replace('0', '') }}
              </div>
            </template>
          </div>
          <span>
            <template v-if="item.stock">
              <v-flex v-for="st in parseStockTooltip(item)" :key="st.L">
                {{ st.L }}:
                {{ st.R == 1 ? $t('Reserved') : st.Q }}
                <br />
              </v-flex>
            </template>
          </span>
        </v-tooltip>
        <v-badge class="btn" color="red" overlap>
          <template v-if="count" slot="badge">
            {{ count }}
          </template>

          <!-- <v-btn flat class="buyButton" small color="success" @click.prevent="buy(item)">
            <v-icon>mdi-cart</v-icon>
          </v-btn>-->
          <v-btn
            v-if="!clear"
            small
            flat
            class="buyButton"
            color="success"
            @click.prevent.stop="buy(item)"
          >
            <input
              v-show="!count && input"
              v-model="qty"
              type="text"
              class="ui-qty"
              @keypress="isNumber($event)"
              @keypress.enter="buy(item)"
              @click.prevent.stop
            />

            <v-icon>mdi-cart</v-icon>
          </v-btn>
        </v-badge>

        <v-menu offset-y left z-index="20">
          <template v-slot:activator="{ on }">
            <v-btn dark small icon class="ui-item-menu" v-on="on">
              <v-icon>mdi-dots-vertical</v-icon>
            </v-btn>
          </template>
          <v-list class="ui-list" slim>
            <v-list-tile @click.prevent="addToFavourite(item)">
              <v-list-tile-action>
                <v-icon v-if="itemQuantity(item.itemNo) > 0" color="orange">
                  mdi-bookmark
                </v-icon>
                <v-icon v-else>
                  mdi-bookmark-outline
                </v-icon>
              </v-list-tile-action>
              <v-list-tile-content>
                {{ $t('Add to favourites') }}
              </v-list-tile-content>
            </v-list-tile>
            <v-list-tile v-if="!garageMode" @click.prevent="dialog = true">
              <v-list-tile-action>
                <v-icon>mdi-garage-variant</v-icon>
              </v-list-tile-action>
              <v-list-tile-content>{{ $t('To Garage') }}</v-list-tile-content>
            </v-list-tile>
            <v-list-tile
              v-if="userInRole('TranslateManager', 'Admin')"
              :to="
                `/admin/analytics/itemcard/${encodeURIComponent(item.itemNo)}`
              "
              target="_blank"
            >
              <v-list-tile-action>
                <v-icon>mdi-open-in-new</v-icon>
              </v-list-tile-action>
              <v-list-tile-content>{{ $t('Edit') }}</v-list-tile-content>
            </v-list-tile>
            <v-list-tile
              v-if="garageMode"
              @click.prevent="$emit('remove-from-garage', item.itemNo)"
            >
              <v-list-tile-action>
                <v-icon>mdi-close</v-icon>
              </v-list-tile-action>
              <v-list-tile-content>
                {{ $t('Delete') }}
              </v-list-tile-content>
            </v-list-tile>
          </v-list>
        </v-menu>
      </div>

      <div class="item-data-hover">
        <v-divider />
        <div>
          <!-- <b>{{ item.itemNo }}</b>
          : {{ item.brand }}-->
          <template v-if="item.discontinued">
            <v-icon
              color="amber darken-4"
              :data-title="$t('Item selling was discontinued')"
            >
              mdi-cancel
            </v-icon>
            {{ $t('Item discontinued') }}
            <br />
          </template>
          <small>{{ item.searchDescription }}</small>
        </div>
      </div>
    </div>
    <template>
      <v-dialog v-if="dialog" v-model="dialog" scrollable max-width="750">
        <garage-dialog :item="item" @closedialog="closeDlg()" />
      </v-dialog>
    </template>
  </div>
</template>

<script>
import api from '../../store/api'
import { mapActions, mapGetters, mapState } from 'vuex'
import garageDialog from '../user/garage/GarageDialog.vue'
import GarageIcon from '../user/garage/GarageIcon.vue'
// import OfferIcon from '@/components/content/OfferIcon.vue'
import OfferIcons from '../content/OfferIcons.vue'

export default {
  components: {
    garageDialog,
    GarageIcon,
    // OfferIcon,
    OfferIcons
  },
  props: {
    item: { type: Object, default: () => null },
    buyAction: { type: Function, default: () => {} },
    url: { type: Function, default: () => '' },
    itemCount: { type: Number, default: () => 0 },
    clear: { type: Boolean, default: () => false },
    input: { type: Boolean, default: () => false },
    garageMode: { type: Boolean, default: () => false }
  },
  data: () => ({
    dialog: false,
    btn1: true,
    qty: '',
    show: false,
    showPrice: false
  }),
  computed: {
    salesOrderMultiple() {
      return this.item.salesOrderMultiple
    },
    sale() {
      return this.item.discGroup == 'РАСПРОДАЖА'
    },
    action() {
      return this.item.discGroup == 'АКЦИЯ'
    },
    discount() {
      return this.item.discGroup == 'УЦЕНКА'
    },
    specprice() {
      return this.specPriceItems[this.item.itemNo] ? true : false //СПЕЦЦЕНА
    },
    ...mapGetters({
      itemQuantity: 'noteModule/itemCount'
    }),
    ...mapState({
      roles: state => state.user.user.roles,
      specPriceItems: state => state.content.specPriceItems
    }),
    count: {
      get() {
        return this.itemCount || ''
      }
    }
  },
  watch: {
    item: {
      handler: function(val) {
        if (val && !val.inAction)
          this.loadInAction({
            data: [val.itemNo],
            then: r => {
              val.inAction = r.data[val.itemNo] || []
            }
          })
      },
      immediate: true
    }
  },
  created() {
    if (
      (!this.specPriceItems || !this.specPriceItems.length) &&
      !this.$store.state.content.pending.specPriceItems
    )
      this.loadInSpecPrice({ data: [] })
  },
  methods: {
    ...mapActions({
      addItem: 'noteModule/addItem',
      loadInAction: 'content/loadInAction',
      removeItem: 'noteModule/removeItem',
      loadInSpecPrice: 'content/loadInSpecPrice'
    }),
    addToFavourite(item) {
      if (this.itemQuantity(item.itemNo) > 0)
        this.removeItem({
          itemNo: item.itemNo
        })
      else
        this.addItem({
          itemNo: item.itemNo,
          quantity: item.qty || 1,
          sourceCode: this.$route.path,
          context: this
        })
    },
    parseStock(item) {
      return api.parseStock(item).slice(0, 6)
    },
    parseStockTooltip(item) {
      var stock = api.parseStock(item)
      var qtyOther = (stock.length > 6 ? stock.slice(6) : [])
        .map(x => x.Q)
        .reduce((acc, item) => {
          return acc + (item === '>9' ? 9 : +item)
        }, 0)
      return [
        ...stock.slice(0, stock.length > 5 ? 6 : stock.length),
        {
          L: this.$t('On other warehouses'),
          Q: qtyOther > 9 ? '>9' : qtyOther
        }
      ]
    },
    imgSrc(item) {
      if (item.firstPic != null)
        return (
          this.$config.catImgUrl + item.firstPic.replace('tcd/', 'tcd-pic/')
        )
      else return this.$config.noPic
    },
    getPriceDecimal(price) {
      if (price) {
        return price
          .toFixed(2)
          .toString()
          .split('.')[1]
      }
    },
    buy(item) {
      if (!this.count) {
        item.qty =
          this.qty.trim() === ''
            ? this.salesOrderMultiple
            : this.checkQuantity(this.qty)
        this.buyAction(item)
      } else {
        this.$emit('remove', item)
        this.qty = ''
      }
    },
    checkQuantity(q) {
      if (q % this.salesOrderMultiple > 0) {
        this.qty =
          Math.ceil(Number(q) / this.salesOrderMultiple) *
          this.salesOrderMultiple
        return this.qty
      } else {
        if (Number(q) == 0) {
          this.qty = this.salesOrderMultiple
          return this.salesOrderMultiple
        } else return Number(q)
      }
    },
    isNumber: function(evt) {
      evt = evt ? evt : window.event
      var charCode = evt.which ? evt.which : evt.keyCode
      if (charCode > 31 && (charCode < 48 || charCode > 57)) {
        evt.preventDefault()
      } else {
        return true
      }
    },
    closeDlg() {
      this.dialog = false
    },
    userInRole(role) {
      if (arguments.length == 1) {
        if (role.__proto__.constructor.name == 'Array') {
          return !role.length || this.roles.some(x => role.includes(x))
        } else {
          return this.roles.some(x => x == role)
        }
      } //if (arguments.length > 0)
      else
        return this.roles.some(x =>
          [].__proto__.slice.apply(arguments).includes(x)
        )
    }
  }
}
</script>
