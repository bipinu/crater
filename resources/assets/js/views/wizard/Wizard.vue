<template>
  <div
    class="flex flex-col items-center justify-between w-full h-32 pt-10 step-indicator"
  >
    <img
      id="logo-crater"
      src="/assets/img/crater-logo.png"
      alt="Crater Logo"
      class="h-12"
    />
    <sw-wizard
      :steps="7"
      :current-step.sync="step"
      :allow-navigation-redirect="false"
    >
      <component :is="tab" @next="setTab" />
    </sw-wizard>
  </div>
</template>

<script>
import SystemRequirement from './WizardSystemRequirementStep'
import Permission from './WizardPermissionStep'
import Database from './WizardDatabaseStep'
import EmailConfiguration from './WizardEmailConfigStep'
import UserProfile from './WizardUserProfileStep'
import CompanyInfo from './WizardCompanyInfoStep'
import Settings from './WizardSettingsStep'

export default {
  components: {
    step_1: SystemRequirement,
    step_2: Permission,
    step_3: Database,
    step_4: EmailConfiguration,
    step_5: UserProfile,
    step_6: CompanyInfo,
    step_7: Settings,
  },
  data() {
    return {
      profile_complete: null,
      loading: false,
      tab: 'step_1',
      step: 1,
    }
  },
  created() {
    this.getProfileComplete()
  },
  methods: {
    async getProfileComplete() {
      let response = await axios.get('/api/v1/onboarding/wizard-step')

      if (response.data.profile_complete === 'COMPLETED') {
        this.$router.push('/admin/dashboard')

        return
      }

      let dbStep = parseInt(response.data.profile_complete)

      if (dbStep) {
        this.step = dbStep + 1
        this.tab = `step_${dbStep + 1}`
      }
    },
    async setProfileComplete(data) {
      let status = {
        profile_complete: data,
      }

      let response = await axios.post('/api/v1/onboarding/wizard-step', status)
    },
    async setTab(data) {
      if (data) {
        this.setProfileComplete(data)
      }
      this.step++

      if (this.step <= 7) {
        this.tab = 'step_' + this.step
      } else {
        // window.location.reload()
      }
    },
  },
}
</script>
