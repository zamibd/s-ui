<template>
  <div class="minimal-login">
    <div class="particles-bg"></div>
    
    <v-container class="fill-height">
      <v-row justify="center" align="center" class="fill-height">
        <v-col cols="12" sm="10" md="6" lg="4">
          <div class="login-container">
            <!-- Header -->
            <div class="login-header text-center mb-10 mb-sm-8">
              <div class="logo-wrapper mb-4">
                <v-avatar 
                  :size="$vuetify.display.xs ? 60 : $vuetify.display.sm ? 70 : 80" 
                  color="primary" 
                  class="elevation-8 responsive-logo"
                >
                  <v-icon 
                    :size="$vuetify.display.xs ? 30 : $vuetify.display.sm ? 35 : 40" 
                    color="white"
                  >
                    mdi-shield-check
                  </v-icon>
                </v-avatar>
              </div>
              <h1 class="responsive-title font-weight-light mb-2">Admin Panel</h1>
              <p class="responsive-subtitle text-medium-emphasis">Secure Panel Access</p>
            </div>

            <!-- Login Card -->
            <v-card 
              class="login-card pa-6 pa-sm-8" 
              flat 
              color="transparent"
            >
              <v-form @submit.prevent="login" ref="form">
                <div class="input-group mb-6 mb-sm-4">
                  <label class="input-label">{{ $t('login.username') }}</label>
                  <v-text-field
                    v-model="username"
                    :rules="usernameRules"
                    variant="solo-filled"
                    flat
                    hide-details="auto"
                    class="minimal-input mobile-friendly-input"
                    required
                    autocomplete="username"
                  ></v-text-field>
                </div>
                
                <div class="input-group mb-8 mb-sm-6">
                  <label class="input-label">{{ $t('login.password') }}</label>
                  <v-text-field
                    v-model="password"
                    :rules="passwordRules"
                    type="password"
                    variant="solo-filled"
                    flat
                    hide-details="auto"
                    class="minimal-input mobile-friendly-input"
                    required
                    autocomplete="current-password"
                  ></v-text-field>
                </div>
                
                <v-btn
                  :loading="loading"
                  type="submit"
                  color="primary"
                  size="large"
                  flat
                  block
                  class="login-submit-btn mb-8 mb-sm-6 mobile-friendly-btn"
                >
                  {{ $t('actions.submit') }}
                  <v-icon end>mdi-arrow-right</v-icon>
                </v-btn>
              </v-form>

              <!-- Footer Controls -->
              <div class="footer-controls d-flex justify-center align-center flex-column flex-sm-row">
                <v-btn-toggle
                  v-model="selectedTheme"
                  mandatory
                  variant="outlined"
                  divided
                  class="mb-4 mb-sm-0 me-sm-4"
                  density="comfortable"
                >
                  <v-btn 
                    v-for="th in themes" 
                    :key="th.value"
                    :value="th.value"
                    size="small"
                    @click="changeTheme(th.value)"
                  >
                    <v-icon size="16">{{ th.icon }}</v-icon>
                  </v-btn>
                </v-btn-toggle>
                
                <v-select
                  :items="languages"
                  v-model="$i18n.locale"
                  @update:modelValue="changeLocale"
                  variant="outlined"
                  density="comfortable"
                  hide-details
                  class="mobile-friendly-select"
                  style="max-width: 100px"
                ></v-select>
              </div>
            </v-card>
          </div>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>

<script lang="ts" setup>
import { ref, computed } from "vue"
import { useLocale, useTheme } from 'vuetify'
import { i18n, languages } from '@/locales'
import { useRouter } from 'vue-router'
import HttpUtil from '@/plugins/httputil'

const theme = useTheme()
const locale = useLocale()

const themes = [
  { value: 'light', icon: 'mdi-white-balance-sunny' },
  { value: 'dark', icon: 'mdi-moon-waning-crescent' },
  { value: 'system', icon: 'mdi-laptop' },
]

const username = ref('')
const usernameRules = [
  (value: string) => {
    if (value?.length > 0) return true
    return i18n.global.t('login.unRules')
  },
]

const password = ref('')
const passwordRules = [
  (value: string) => {
    if (value?.length > 0) return true
    return i18n.global.t('login.pwRules')
  },
]

const loading = ref(false)
const router = useRouter()

const login = async () => {
  if (username.value == '' || password.value == '') return
  loading.value = true
  const response = await HttpUtil.post('api/login', { user: username.value, pass: password.value })
  if (response.success) {
    setTimeout(() => {
      loading.value = false
      router.push('/')
    }, 500)
  } else {
    loading.value = false
  }
}
const changeLocale = (l: any) => {
  locale.current.value = l ?? 'en'
  localStorage.setItem('locale', locale.current.value)
}
const changeTheme = (th: string) => {
  theme.change(th)
  localStorage.setItem('theme', th)
}
const isActiveTheme = (th: string) => {
  const current = localStorage.getItem('theme') ?? 'system'
  return current == th
}

const selectedTheme = computed({
  get: () => localStorage.getItem('theme') ?? 'system',
  set: () => {}
})
</script>

<style scoped>
.minimal-login {
  min-height: 100vh;
  background: #0a0a0a;
  position: relative;
  overflow: hidden;
}

.particles-bg {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-image: 
    radial-gradient(2px 2px at 20px 30px, #ffffff10, transparent),
    radial-gradient(2px 2px at 40px 70px, #ffffff08, transparent),
    radial-gradient(1px 1px at 90px 40px, #ffffff15, transparent),
    radial-gradient(1px 1px at 130px 80px, #ffffff10, transparent);
  background-repeat: repeat;
  background-size: 200px 200px;
  animation: particleMove 20s linear infinite;
}

@keyframes particleMove {
  0% { transform: translate(0, 0); }
  100% { transform: translate(-200px, -200px); }
}

.login-container {
  position: relative;
  z-index: 1;
}

.logo-wrapper {
  animation: logoFloat 3s ease-in-out infinite;
}

@keyframes logoFloat {
  0%, 100% { transform: translateY(0px); }
  50% { transform: translateY(-10px); }
}

.login-header h1 {
  color: white;
  letter-spacing: 3px;
}

.input-group {
  position: relative;
}

.input-label {
  display: block;
  color: #ffffff80;
  font-size: 0.875rem;
  font-weight: 500;
  margin-bottom: 8px;
  text-transform: uppercase;
  letter-spacing: 1px;
}

.minimal-input :deep(.v-field) {
  background: rgba(255, 255, 255, 0.05) !important;
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 4px !important;
  transition: all 0.3s ease;
}

.minimal-input :deep(.v-field):hover {
  background: rgba(255, 255, 255, 0.08) !important;
  border-color: rgba(255, 255, 255, 0.2);
}

.minimal-input :deep(.v-field--focused) {
  background: rgba(255, 255, 255, 0.1) !important;
  border-color: rgb(var(--v-theme-primary)) !important;
  box-shadow: 0 0 0 1px rgb(var(--v-theme-primary));
}

.login-submit-btn {
  background: linear-gradient(45deg, rgb(var(--v-theme-primary)), rgb(var(--v-theme-secondary))) !important;
  text-transform: none;
  font-weight: 600;
  letter-spacing: 1px;
  transition: all 0.3s ease;
}

.login-submit-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 25px rgba(var(--v-theme-primary), 0.3);
}

.footer-controls {
  gap: 16px;
}

/* Light theme adjustments */
.v-theme--light .minimal-login {
  background: #f8f9fa;
}

.v-theme--light .particles-bg {
  background-image: 
    radial-gradient(2px 2px at 20px 30px, #00000010, transparent),
    radial-gradient(2px 2px at 40px 70px, #00000008, transparent),
    radial-gradient(1px 1px at 90px 40px, #00000015, transparent),
    radial-gradient(1px 1px at 130px 80px, #00000010, transparent);
}

.v-theme--light .login-header h1 {
  color: #1a1a1a;
}

.v-theme--light .input-label {
  color: #666;
}

.v-theme--light .minimal-input :deep(.v-field) {
  background: rgba(0, 0, 0, 0.02) !important;
  border-color: rgba(0, 0, 0, 0.1);
}

.v-theme--light .minimal-input :deep(.v-field):hover {
  background: rgba(0, 0, 0, 0.04) !important;
  border-color: rgba(0, 0, 0, 0.2);
}

/* Mobile-specific styles */
@media (max-width: 600px) {
  .login-container {
    padding: 0 16px;
  }

  .login-card {
    padding: 24px 20px !important;
  }

  .particles-bg {
    /* Reduce animation intensity on mobile for better performance */
    animation-duration: 30s;
    opacity: 0.7;
  }

  .mobile-friendly-input :deep(.v-field) {
    font-size: 16px; /* Prevents zoom on iOS */
    min-height: 48px; /* Touch-friendly height */
  }

  .mobile-friendly-input :deep(.v-field__input) {
    padding: 12px 16px;
  }

  .mobile-friendly-btn {
    font-size: 16px;
    min-height: 48px; /* Touch-friendly height */
    padding: 12px 16px;
  }

  .mobile-friendly-select :deep(.v-select__selections) {
    font-size: 14px;
  }

  .mobile-friendly-select :deep(.v-field) {
    min-height: 40px;
  }

  /* Responsive typography */
  .responsive-title {
    font-size: 1.5rem !important;
    line-height: 1.3;
  }

  .responsive-subtitle {
    font-size: 0.875rem !important;
  }

  /* Better touch targets for theme toggle */
  .footer-controls .v-btn-toggle .v-btn {
    min-width: 44px;
    min-height: 44px;
  }

  /* Reduce margins on mobile */
  .input-group {
    margin-bottom: 20px !important;
  }

  .login-submit-btn {
    margin-bottom: 24px !important;
  }
}

/* Tablet styles */
@media (min-width: 601px) and (max-width: 960px) {
  .responsive-title {
    font-size: 1.75rem !important;
  }

  .responsive-subtitle {
    font-size: 1rem !important;
  }
}

/* Desktop styles */
@media (min-width: 961px) {
  .responsive-title {
    font-size: 2rem !important;
  }

  .responsive-subtitle {
    font-size: 1.125rem !important;
  }
}

/* High DPI displays */
@media (-webkit-min-device-pixel-ratio: 2), (min-resolution: 192dpi) {
  .responsive-logo {
    /* Ensure crisp icons on retina displays */
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
  }
}
</style>
