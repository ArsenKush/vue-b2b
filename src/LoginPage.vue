<template>
  <v-layout row wrap>
    <v-dialog v-model="dialog" persistent max-width="300">
      <form @submit.prevent="handleSubmit">
        <v-card>
          <v-toolbar dark color="primary" dense>
            <!-- <img src="/img/logo.png" style="max-height: 100%;margin-right:8px; padding: 2px 0;"> -->
            <img
              src="@/assets/img/logo.svg"
              style="width: 75px; height: 50px;"
            />

            <v-toolbar-title>{{ $t('Auth') }}</v-toolbar-title>
          </v-toolbar>
          <v-card-text class="pb-0">
            <v-select
              v-show="companies.length > 1"
              v-model="selectedCompany"
              :label="$t('Company name')"
              autocomplete:true
              :items="companies"
              item-text="name"
              item-value="comId"
            />
            <v-text-field
              v-model="login"
              :label="$t('Login')"
              name="Login"
              placeholder=" "
              type="text"
              required
              :rules="requiredRule"
            />
            <v-text-field
              v-model="pass"
              :label="$t('Password')"
              name="Password"
              placeholder=" "
              type="password"
              required
              :rules="requiredRule"
            />
          </v-card-text>
          <v-card-actions class="px-3 pt-0">
            <!-- <v-spacer /> -->
            <v-btn color="primary" type="submit" block>
              {{ $t('Enter') }}
            </v-btn>
            <!-- <v-btn color="green darken-1" flat type="submit">
              {{ $t('Enter') }}
            </v-btn> -->
          </v-card-actions>

          <center class="pb-2">
            <router-link to="/forgot">
              {{ $t('Forgot password') }}
            </router-link>
            <br />

            <!-- <router-link to="/register">
              {{ $t('Registration') }}
            </router-link> -->
            <!-- <v-spacer /> -->
          </center>
          <!-- <v-spacer /> -->
          <!-- <v-btn color="grey darken-1" flat type="reset">
              {{ $t('Clear') }}
            </v-btn> -->
        </v-card>
      </form>
    </v-dialog>
  </v-layout>
</template>

<script>
import { mapState, mapActions } from 'vuex'

export default {
  data: () => ({
    login: '',
    pass: '',
    dialog: true
  }),

  computed: {
    requiredRule() {
      return [v => !!v || this.$t('This is a required field')]
    },
    selectedCompany: {
      get() {
        return this.comId || (this.companies[0] ? this.companies[0].comId : 15)
      },
      set(value) {
        this.setComId(value)
      }
    },
    ...mapState({
      token: state => state.user.user.token,
      comId: state => state.user.user.comId,
      companies: state => state.company.companies
    })
  },
  created: function() {
    if (this.token) this.$router.push('/')

    this.getActiveCompanies({
      then: companies => {
        if (companies.length == 1) this.setComId(companies[0].comId)
      }
    })
  },
  methods: {
    ...mapActions({
      setLogin: 'user/login',
      setComId: 'user/setComId',
      loadSearchModule: 'searchModule/loadSearchModule',
      loadCartModule: 'cartModule/loadModule',
      loadGarage: 'garageModule/loadGarage',
      loadNoteModule: 'noteModule/loadModule',
      setSnackbar: 'snackbarModule/setSnackbar',
      unreadBugCount: 'common/unreadBugCount',
      setIntervalFunc: 'user/setIntervalFunc',
      getActiveCompanies: 'company/allCompanies'
    }),
    handleSubmit() {
      if (!this.login || !this.pass)
        this.setSnackbar({
          color: 'error',
          text: this.$t('Fill in all required fields')
        })
      else {
        this.setLogin({
          data: {
            comId: this.selectedCompany,
            login: this.login,
            pwd: this.pass
          },
          params: {
            then: () => {
              this.dialog = false
              this.loadSearchModule()
              this.loadCartModule()
              this.loadNoteModule()
              this.loadGarage({
                data: this.$enums.setType.garage
              })
              /* start reading new bugs*/
              this.setIntervalFunc({
                func: this.unreadBugCount,
                interval: 60000
              })
              /* end */
            },
            error: error => {
              this.setSnackbar({
                color: 'error',
                text: this.$t(error.response.data)
              })
            }
          }
        })
      }
    }
  }
}
</script>
