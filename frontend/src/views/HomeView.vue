<template>
  <!-- Custom Home Content: Full Page Mode -->
  <div v-if="homeContent" class="min-h-screen">
    <!-- iframe mode -->
    <iframe
      v-if="isHomeContentUrl"
      :src="homeContent.trim()"
      class="h-screen w-full border-0"
      allowfullscreen
    ></iframe>
    <!-- HTML mode - SECURITY: homeContent is admin-only setting, XSS risk is acceptable -->
    <div v-else v-html="homeContent"></div>
  </div>

  <!-- Default Landing Page -->
  <div
    v-else
    class="landing-page relative min-h-screen overflow-hidden"
  >
    <div class="pointer-events-none absolute inset-0 overflow-hidden">
      <div class="landing-grid absolute inset-0"></div>
      <div class="landing-wash absolute inset-0"></div>
      <div class="scan-line absolute left-0 top-0 h-px w-full"></div>
    </div>

    <!-- Header -->
    <header class="relative z-20 px-4 py-4 sm:px-6">
      <nav class="mx-auto flex max-w-7xl items-center justify-between gap-4">
        <router-link to="/home" class="flex min-w-0 items-center gap-3">
          <div class="brand-mark h-10 w-10 flex-none overflow-hidden rounded-lg p-1">
            <img :src="siteLogo || '/logo.png'" alt="Logo" class="h-full w-full object-contain" />
          </div>
          <span class="landing-brand truncate text-sm font-semibold tracking-wide sm:text-base">{{ siteName }}</span>
        </router-link>

        <div class="flex items-center gap-2 sm:gap-3">
          <LocaleSwitcher />

          <a
            v-if="docUrl"
            :href="docUrl"
            target="_blank"
            rel="noopener noreferrer"
            class="landing-icon-button"
            :title="t('home.viewDocs')"
          >
            <Icon name="book" size="md" />
          </a>

          <button
            @click="toggleTheme"
            class="landing-icon-button"
            :title="isDark ? t('home.switchToLight') : t('home.switchToDark')"
          >
            <Icon v-if="isDark" name="sun" size="md" />
            <Icon v-else name="moon" size="md" />
          </button>

          <router-link
            v-if="isAuthenticated"
            :to="dashboardPath"
            class="dashboard-pill hidden items-center gap-1.5 rounded-full py-1 pl-1 pr-3 text-xs font-medium transition sm:inline-flex"
          >
            <span class="flex h-6 w-6 items-center justify-center rounded-full bg-gradient-to-br from-violet-500 to-fuchsia-500 text-[10px] font-semibold text-white">
              {{ userInitial }}
            </span>
            {{ t('home.dashboard') }}
          </router-link>
          <template v-else>
            <router-link
              to="/login"
              class="login-link hidden rounded-full px-3 py-2 text-xs font-medium transition sm:inline-flex"
            >
              {{ t('home.login') }}
            </router-link>
            <router-link
              to="/register"
              class="register-pill inline-flex rounded-full px-4 py-2 text-xs font-semibold transition"
            >
              {{ copy.navRegister }}
            </router-link>
          </template>
        </div>
      </nav>
    </header>

    <main class="relative z-10">
      <!-- Hero -->
      <section class="px-4 pb-16 pt-12 sm:px-6 sm:pb-20 lg:pb-24 lg:pt-20">
        <div class="mx-auto grid max-w-7xl items-center gap-12 lg:grid-cols-[1.02fr_0.98fr]">
          <div class="max-w-3xl text-center lg:text-left">
            <div class="hero-badge mb-5 inline-flex items-center gap-2 rounded-full px-3 py-1 text-xs font-medium backdrop-blur">
              <Icon name="sparkles" size="sm" class="text-fuchsia-300" />
              {{ copy.heroBadge }}
            </div>

            <h1 class="hero-title text-balance text-4xl font-black leading-[1.18] sm:text-5xl lg:text-7xl">
              {{ copy.heroTitleA }}
              <span class="hero-title-gradient block bg-clip-text text-transparent">
                {{ copy.heroTitleB }}
              </span>
            </h1>

            <p class="landing-muted mx-auto mt-6 max-w-2xl text-base leading-8 sm:text-lg lg:mx-0">
              {{ copy.heroDescription }}
            </p>

            <div class="mt-8 flex flex-col justify-center gap-3 sm:flex-row lg:justify-start">
              <router-link :to="primaryCtaPath" class="landing-primary-button">
                {{ isAuthenticated ? t('home.goToDashboard') : copy.primaryCta }}
                <Icon name="arrowRight" size="md" :stroke-width="2" />
              </router-link>
              <router-link :to="apiKeyCtaPath" class="landing-secondary-button">
                <Icon name="key" size="md" />
                {{ copy.apiKeyCta }}
              </router-link>
            </div>

            <div class="mt-8 grid grid-cols-3 gap-3 text-left">
              <div v-for="metric in copy.heroMetrics" :key="metric.label" class="metric-card rounded-lg p-3 backdrop-blur">
                <div class="landing-strong text-xl font-bold sm:text-2xl">{{ metric.value }}</div>
                <div class="landing-subtle mt-1 text-xs leading-5">{{ metric.label }}</div>
              </div>
            </div>
          </div>

          <div class="relative mx-auto w-full max-w-xl">
            <div class="api-visual">
              <div class="api-visual-header">
                <div class="flex items-center gap-2">
                  <span class="h-2.5 w-2.5 rounded-full bg-rose-400"></span>
                  <span class="h-2.5 w-2.5 rounded-full bg-amber-300"></span>
                  <span class="h-2.5 w-2.5 rounded-full bg-emerald-400"></span>
                </div>
                <span>{{ copy.visualTitle }}</span>
              </div>
              <div class="api-route-map">
                <div class="route-node route-node-user">
                  <Icon name="terminal" size="lg" />
                  <span>{{ copy.visualClient }}</span>
                </div>
                <div class="route-line"></div>
                <div class="route-node route-node-gateway">
                  <Icon name="server" size="lg" />
                  <span>{{ copy.visualGateway }}</span>
                </div>
                <div class="route-line"></div>
                <div class="grid gap-2">
                  <div v-for="provider in copy.visualProviders" :key="provider" class="provider-chip">
                    <span class="h-2 w-2 rounded-full bg-emerald-300"></span>
                    {{ provider }}
                  </div>
                </div>
              </div>
              <div class="code-panel">
                <div class="code-panel-header flex items-center justify-between px-4 py-3 text-xs">
                  <span>OpenAI compatible</span>
                  <span class="text-emerald-300">200 OK</span>
                </div>
                <pre><code>const openai = new OpenAI({
  apiKey: "{{ maskedApiKey }}",
  baseURL: "{{ apiBaseUrl }}/v1"
})</code></pre>
              </div>
              <div class="api-activity">
                <div v-for="item in copy.visualActivity" :key="item.label" class="activity-row">
                  <span>{{ item.label }}</span>
                  <strong>{{ item.value }}</strong>
                </div>
              </div>
            </div>
          </div>
        </div>
      </section>

      <!-- Advantages -->
      <section class="landing-band px-4 py-16 sm:px-6">
        <div class="mx-auto max-w-7xl">
          <div class="section-heading">
            <p>{{ copy.advantagesEyebrow }}</p>
            <h2>{{ copy.advantagesTitle }}</h2>
          </div>
          <div class="mt-10 grid gap-4 sm:grid-cols-2 lg:grid-cols-4">
            <article v-for="advantage in copy.advantages" :key="advantage.title" class="landing-card">
              <div class="mb-5 flex h-11 w-11 items-center justify-center rounded-lg" :class="advantage.iconClass">
                <Icon :name="advantage.icon" size="lg" />
              </div>
              <h3 class="landing-strong text-lg font-semibold">{{ advantage.title }}</h3>
              <p class="landing-muted mt-3 text-sm leading-7">{{ advantage.description }}</p>
            </article>
          </div>
        </div>
      </section>

      <!-- Scenarios -->
      <section class="px-4 py-16 sm:px-6 lg:py-20">
        <div class="mx-auto grid max-w-7xl gap-10 lg:grid-cols-[0.82fr_1.18fr] lg:items-start">
          <div class="section-heading text-left">
            <p>{{ copy.scenariosEyebrow }}</p>
            <h2>{{ copy.scenariosTitle }}</h2>
            <span>{{ copy.scenariosDescription }}</span>
          </div>
          <div class="grid gap-3 sm:grid-cols-2">
            <div v-for="scenario in copy.scenarios" :key="scenario.title" class="scenario-row">
              <Icon :name="scenario.icon" size="md" class="text-fuchsia-300" />
              <div>
                <h3 class="landing-strong font-semibold">{{ scenario.title }}</h3>
                <p class="landing-muted mt-1 text-sm leading-6">{{ scenario.description }}</p>
              </div>
            </div>
          </div>
        </div>
      </section>

      <!-- Integration -->
      <section class="integration-band px-4 py-16 sm:px-6 lg:py-20">
        <div class="mx-auto grid max-w-7xl gap-8 lg:grid-cols-[0.95fr_1.05fr] lg:items-center">
          <div>
            <div class="section-heading text-left">
              <p>{{ copy.integrationEyebrow }}</p>
              <h2>{{ copy.integrationTitle }}</h2>
              <span>{{ copy.integrationDescription }}</span>
            </div>
            <div class="mt-8 grid gap-3">
              <div v-for="step in copy.integrationSteps" :key="step.title" class="step-row flex gap-4 rounded-lg p-4">
                <div class="step-index flex h-8 w-8 flex-none items-center justify-center rounded-lg text-sm font-bold">
                  {{ step.index }}
                </div>
                <div>
                  <h3 class="landing-strong font-semibold">{{ step.title }}</h3>
                  <p class="landing-muted mt-1 text-sm leading-6">{{ step.description }}</p>
                </div>
              </div>
            </div>
          </div>
          <div class="code-showcase">
            <div class="code-showcase-header flex items-center justify-between px-4 py-3">
              <div class="landing-strong flex items-center gap-2 text-sm font-medium">
                <Icon name="terminal" size="sm" />
                {{ copy.codeTitle }}
              </div>
              <span class="rounded-full bg-emerald-400/10 px-2 py-1 text-xs text-emerald-300">{{ copy.compatible }}</span>
            </div>
            <pre><code>curl {{ apiBaseUrl }}/v1/chat/completions \
  -H "Authorization: Bearer {{ maskedApiKey }}" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4o",
    "messages": [
      { "role": "user", "content": "Hello API Gateway" }
    ]
  }'</code></pre>
          </div>
        </div>
      </section>

      <!-- Pricing -->
      <section class="px-4 py-16 sm:px-6 lg:py-20">
        <div class="mx-auto max-w-7xl">
          <div class="section-heading">
            <p>{{ copy.pricingEyebrow }}</p>
            <h2>{{ copy.pricingTitle }}</h2>
            <span>{{ copy.pricingDescription }}</span>
          </div>
          <div class="mt-10 grid gap-4 lg:grid-cols-2">
            <article v-for="plan in copy.plans" :key="plan.name" class="pricing-card" :class="{ 'pricing-card-featured': plan.featured }">
              <div class="flex flex-col gap-4 sm:flex-row sm:items-start sm:justify-between">
                <div>
                  <div class="flex items-center gap-3">
                    <h3 class="landing-strong text-2xl font-bold">{{ plan.name }}</h3>
                    <span v-if="plan.badge" class="plan-badge rounded-full px-3 py-1 text-xs font-semibold">{{ plan.badge }}</span>
                  </div>
                  <p class="landing-muted mt-3 text-sm leading-7">{{ plan.description }}</p>
                </div>
                <div class="text-left sm:text-right">
                  <div class="landing-strong text-3xl font-black">{{ plan.price }}</div>
                  <div class="landing-subtle text-xs">{{ plan.unit }}</div>
                </div>
              </div>
              <div class="mt-6 grid gap-3 sm:grid-cols-2">
                <div v-for="feature in plan.features" :key="feature" class="landing-body flex items-center gap-2 text-sm">
                  <Icon name="checkCircle" size="sm" class="text-emerald-300" />
                  {{ feature }}
                </div>
              </div>
              <router-link :to="purchaseCtaPath" class="plan-button mt-7 inline-flex w-full items-center justify-center gap-2 rounded-lg px-4 py-3 text-sm font-bold transition">
                {{ plan.cta }}
                <Icon name="arrowRight" size="sm" :stroke-width="2" />
              </router-link>
            </article>
          </div>
        </div>
      </section>

      <!-- FAQ + Final CTA -->
      <section class="faq-section px-4 py-16 sm:px-6 lg:py-20">
        <div class="mx-auto grid max-w-7xl gap-10 lg:grid-cols-[0.9fr_1.1fr]">
          <div>
            <div class="section-heading text-left">
              <p>{{ copy.faqEyebrow }}</p>
              <h2>{{ copy.faqTitle }}</h2>
              <span>{{ copy.faqDescription }}</span>
            </div>
            <router-link :to="primaryCtaPath" class="landing-primary-button mt-8">
              {{ isAuthenticated ? t('home.goToDashboard') : copy.finalCta }}
              <Icon name="arrowRight" size="md" :stroke-width="2" />
            </router-link>
          </div>
          <div class="grid gap-3">
            <details v-for="faq in copy.faqs" :key="faq.question" class="faq-item">
              <summary>{{ faq.question }}</summary>
              <p>{{ faq.answer }}</p>
            </details>
          </div>
        </div>
      </section>
    </main>

    <!-- Footer -->
    <footer class="landing-footer relative z-10 px-4 py-8 sm:px-6">
      <div class="mx-auto flex max-w-7xl flex-col items-center justify-between gap-4 text-center sm:flex-row sm:text-left">
        <p class="footer-text text-sm">
          &copy; {{ currentYear }} {{ siteName }}. {{ t('home.footer.allRightsReserved') }}
        </p>
        <div class="flex items-center gap-4">
          <a
            v-if="docUrl"
            :href="docUrl"
            target="_blank"
            rel="noopener noreferrer"
            class="footer-link text-sm transition"
          >
            {{ t('home.docs') }}
          </a>
          <a
            :href="githubUrl"
            target="_blank"
            rel="noopener noreferrer"
            class="footer-link text-sm transition"
          >
            GitHub
          </a>
          <router-link v-if="!isAuthenticated" to="/login" class="footer-link text-sm transition">
            {{ t('home.login') }}
          </router-link>
          <router-link v-if="!isAuthenticated" to="/register" class="footer-link text-sm transition">
            {{ copy.navRegister }}
          </router-link>
        </div>
      </div>
    </footer>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import { useI18n } from 'vue-i18n'
import { useAuthStore, useAppStore } from '@/stores'
import LocaleSwitcher from '@/components/common/LocaleSwitcher.vue'
import Icon from '@/components/icons/Icon.vue'

type IconName = InstanceType<typeof Icon>['$props']['name']

interface AdvantageItem {
  title: string
  description: string
  icon: IconName
  iconClass: string
}

interface ScenarioItem {
  title: string
  description: string
  icon: IconName
}

const { t, locale } = useI18n()

const authStore = useAuthStore()
const appStore = useAppStore()

// Site settings - directly from appStore (already initialized from injected config)
const siteName = computed(() => appStore.cachedPublicSettings?.site_name || appStore.siteName || 'Sub2API')
const siteLogo = computed(() => appStore.cachedPublicSettings?.site_logo || appStore.siteLogo || '')
const siteSubtitle = computed(() => appStore.cachedPublicSettings?.site_subtitle || 'AI API Gateway Platform')
const docUrl = computed(() => appStore.cachedPublicSettings?.doc_url || appStore.docUrl || '')
const homeContent = computed(() => appStore.cachedPublicSettings?.home_content || '')

// Check if homeContent is a URL (for iframe display)
const isHomeContentUrl = computed(() => {
  const content = homeContent.value.trim()
  return content.startsWith('http://') || content.startsWith('https://')
})

// Theme
const isDark = ref(document.documentElement.classList.contains('dark'))

// GitHub URL
const githubUrl = 'https://github.com/Wei-Shaw/sub2api'

// Auth state
const isAuthenticated = computed(() => authStore.isAuthenticated)
const isAdmin = computed(() => authStore.isAdmin)
const dashboardPath = computed(() => isAdmin.value ? '/admin/dashboard' : '/dashboard')
const userInitial = computed(() => {
  const user = authStore.user
  if (!user || !user.email) return ''
  return user.email.charAt(0).toUpperCase()
})

const primaryCtaPath = computed(() => isAuthenticated.value ? dashboardPath.value : '/register')
const apiKeyCtaPath = computed(() => isAuthenticated.value ? '/keys' : '/register')
const purchaseCtaPath = computed(() => isAuthenticated.value ? '/purchase' : '/register')
const apiBaseUrl = computed(() => {
  if (typeof window === 'undefined') return 'https://api.example.com'
  return window.location.origin
})
const maskedApiKey = 'sk-...token-flow'

const copy = computed(() => {
  const isZh = String(locale.value).toLowerCase().startsWith('zh')
  if (!isZh) {
    return {
      navRegister: 'Register',
      heroBadge: `${siteName.value} · ${siteSubtitle.value}`,
      heroTitleA: 'Pay as you go.',
      heroTitleB: 'Connect your AI coding tools.',
      heroDescription:
        'Stable API access for Codex, Claude Code, openclaw, Hermes, and other AI tools. No monthly lock-in, no time limits, billed by actual usage so developers and teams can use AI more freely.',
      primaryCta: 'Get API Key',
      apiKeyCta: 'View setup guide',
      heroMetrics: [
        { value: 'Usage', label: 'Pay only for what you use' },
        { value: 'No limit', label: 'Use it whenever you need' },
        { value: 'AI Tools', label: 'Codex and more' }
      ],
      visualTitle: 'AI tool API access',
      visualClient: 'AI tools',
      visualGateway: 'Unified API access',
      visualProviders: ['Codex', 'Claude Code', 'openclaw', 'Hermes'],
      visualActivity: [
        { label: 'Billing', value: 'Usage' },
        { label: 'Access', value: 'No time limit' },
        { label: 'Setup', value: 'Ready to use' }
      ],
      advantagesEyebrow: 'Why it works',
      advantagesTitle: 'Flexible API access for the AI tools you already use',
      advantages: [
        {
          title: 'Pay as you go',
          description: 'No monthly subscription lock-in. Billing follows actual API usage, making both light testing and long-term use more flexible.',
          icon: 'dollar' as IconName,
          iconClass: 'advantage-icon advantage-icon-fuchsia'
        },
        {
          title: 'No time limits',
          description: 'Open your tools and call the API whenever you need. It fits ongoing development, agents, and automation tasks.',
          icon: 'clock' as IconName,
          iconClass: 'advantage-icon advantage-icon-emerald'
        },
        {
          title: 'Use it freely',
          description: 'Register, get an API Key, and connect common AI tools with less setup friction and waiting.',
          icon: 'bolt' as IconName,
          iconClass: 'advantage-icon advantage-icon-cyan'
        },
        {
          title: 'Mainstream tool support',
          description: 'Built for Codex, Claude Code, openclaw, Hermes, and similar tool workflows with unified access and usage management.',
          icon: 'shield' as IconName,
          iconClass: 'advantage-icon advantage-icon-amber'
        }
      ] satisfies AdvantageItem[],
      scenariosEyebrow: 'Use cases',
      scenariosTitle: 'Built around modern AI coding workflows',
      scenariosDescription: 'Use one API Key across mainstream AI tools, personal coding sessions, and team automation workloads.',
      scenarios: [
        { title: 'Codex workflows', description: 'Give Codex a stable API Key for coding, refactoring, and command-line development tasks.', icon: 'cpu' as IconName },
        { title: 'Claude Code', description: 'Connect Claude Code-style workflows with flexible usage-based API access.', icon: 'chat' as IconName },
        { title: 'openclaw', description: 'Use openclaw with a unified entry point for everyday AI-assisted development.', icon: 'sync' as IconName },
        { title: 'Hermes', description: 'Power Hermes workflows without committing to fixed monthly usage windows.', icon: 'document' as IconName },
        { title: 'Personal AI coding', description: 'Pay by usage for experiments, side projects, and daily coding sessions.', icon: 'globe' as IconName },
        { title: 'Team automation', description: 'Support agents and recurring team tasks with unified key and usage management.', icon: 'users' as IconName }
      ] satisfies ScenarioItem[],
      integrationEyebrow: 'Integration',
      integrationTitle: 'Get an API Key and connect your AI tool',
      integrationDescription: 'Use the API address and key required by your tool. The platform handles access, usage records, and billing behind the scenes.',
      integrationSteps: [
        { index: '01', title: 'Register account', description: 'Create an account and enter the console.' },
        { index: '02', title: 'Generate API Key', description: 'Create a key for Codex, Claude Code, openclaw, Hermes, or your team.' },
        { index: '03', title: 'Configure your tool', description: 'Set the API address and key in the tool, then start using it on demand.' }
      ],
      codeTitle: 'Request example',
      compatible: 'OpenAI SDK compatible',
      pricingEyebrow: 'Pricing',
      pricingTitle: 'Usage-based access for different workloads',
      pricingDescription: 'Start with occasional coding sessions, then scale to frequent tool usage and team automation as needed.',
      plans: [
        {
          name: 'Pay as you go',
          badge: '',
          description: 'For personal tools, experiments, and flexible AI coding without a monthly commitment.',
          price: 'Metered',
          unit: 'pay only for actual usage',
          features: ['Usage-based billing', 'No time limits', 'AI tool access', 'Usage dashboard'],
          cta: 'Start metered usage',
          featured: false
        },
        {
          name: 'High-frequency use',
          badge: 'Recommended',
          description: 'For ongoing development, team tools, and automation tasks that use AI throughout the day.',
          price: 'Flexible',
          unit: 'for sustained AI workflows',
          features: ['Frequent tool usage', 'Team-friendly access', 'Automation workflows', 'Unified usage management'],
          cta: 'Use it frequently',
          featured: true
        }
      ],
      faqEyebrow: 'FAQ',
      faqTitle: 'Common questions before you connect',
      faqDescription: 'The short version: no monthly lock-in, no time limit, and mainstream AI tools can connect with an API Key.',
      finalCta: 'Get API Key now',
      faqs: [
        { question: 'Do I need a monthly subscription?', answer: 'No. Billing is based on actual usage, so you only pay for what you use.' },
        { question: 'Is usage limited by time?', answer: 'No. It is not restricted by session duration. Use it whenever you need it.' },
        { question: 'Which AI tools are supported?', answer: 'It supports access scenarios for Codex, Claude Code, openclaw, Hermes, and other mainstream AI tools.' },
        { question: 'How do I start?', answer: 'Register, get an API Key, then configure the API address and key required by your tool.' }
      ]
    }
  }

  return {
    navRegister: '注册',
    heroBadge: `${siteName.value} · ${siteSubtitle.value}`,
    heroTitleA: '按Plus用量定价',
    heroTitleB: '享受 Vibe Coding',
    heroDescription:
      '为 Codex、Claude Code、openclaw、Hermes 等 AI 工具提供稳定 API 接入。无需包月，不限时间，按实际用量计费，想用就用，让个人开发者和团队都能更自由地使用 AI。',
    primaryCta: '立即获取 API Key',
    apiKeyCta: '查看接入方式',
    heroMetrics: [
      { value: '按量计费', label: '用多少算多少' },
      { value: '不限时间', label: '想用就用' },
      { value: 'AI 工具', label: 'Codex 等主流工具' }
    ],
    visualTitle: 'AI 工具 API 接入',
    visualClient: 'AI 工具',
    visualGateway: '统一 API 接入',
    visualProviders: ['Codex', 'Claude Code', 'openclaw', 'Hermes'],
    visualActivity: [
      { label: '计费', value: '按量' },
      { label: '使用', value: '不限时' },
      { label: '接入', value: '即开即用' }
    ],
    advantagesEyebrow: '核心优势',
    advantagesTitle: '为你常用的 AI 工具提供更自由的 API 接入',
    advantages: [
      {
        title: '按量计费',
        description: '无需包月绑定，按实际调用量计费，轻量试用和长期使用都更灵活。',
        icon: 'dollar' as IconName,
        iconClass: 'advantage-icon advantage-icon-fuchsia'
      },
      {
        title: '不限时间使用',
        description: '不按会话时长限制，随时打开工具、随时调用 API，适合持续开发和自动化任务。',
        icon: 'clock' as IconName,
        iconClass: 'advantage-icon advantage-icon-emerald'
      },
      {
        title: '想用就用',
        description: '注册后获取 API Key，即可接入常见 AI 工具，减少复杂配置和等待成本。',
        icon: 'bolt' as IconName,
        iconClass: 'advantage-icon advantage-icon-cyan'
      },
      {
        title: '主流 AI 工具支持',
        description: '支持 Codex、Claude Code、openclaw、Hermes 等工具场景，统一管理接入和用量。',
        icon: 'shield' as IconName,
        iconClass: 'advantage-icon advantage-icon-amber'
      }
    ] satisfies AdvantageItem[],
    scenariosEyebrow: '使用场景',
    scenariosTitle: '围绕主流 AI 编程工具的日常工作流',
    scenariosDescription: '一把 API Key 覆盖 Codex、Claude Code、openclaw、Hermes，以及个人开发和团队自动化场景。',
    scenarios: [
      { title: 'Codex 开发工作流', description: '为 Codex 提供稳定 API Key，覆盖代码生成、重构和命令行开发任务。', icon: 'cpu' as IconName },
      { title: 'Claude Code', description: '用按量计费的 API 接入方式支持 Claude Code 类开发工作流。', icon: 'chat' as IconName },
      { title: 'openclaw', description: '为 openclaw 提供统一入口，适合日常 AI 辅助开发。', icon: 'sync' as IconName },
      { title: 'Hermes', description: '让 Hermes 工作流不再受固定包月和时间窗口限制。', icon: 'document' as IconName },
      { title: '个人 AI 编程', description: '适合实验项目、个人工具和日常编码，用多少算多少。', icon: 'globe' as IconName },
      { title: '团队自动化任务', description: '为 Agent、定时任务和团队流程提供统一密钥与用量管理。', icon: 'users' as IconName }
    ] satisfies ScenarioItem[],
    integrationEyebrow: '接入示例',
    integrationTitle: '获取 API Key，接入你的 AI 工具',
    integrationDescription: '按工具要求配置 API 地址和密钥，平台负责接入、用量统计和按量计费。',
    integrationSteps: [
      { index: '01', title: '注册账户', description: '创建账号并进入控制台。' },
      { index: '02', title: '生成 API Key', description: '为 Codex、Claude Code、openclaw、Hermes 或团队成员创建访问密钥。' },
      { index: '03', title: '配置工具', description: '在工具中填入 API 地址和密钥，即可按需使用。' }
    ],
    codeTitle: '请求示例',
    compatible: '兼容 OpenAI SDK',
    pricingEyebrow: '价格方案',
    pricingTitle: '按实际使用量付费，轻量和高频都灵活',
    pricingDescription: '偶尔使用时无需包月，高频开发和团队自动化也可以按实际调用量持续使用。',
    plans: [
      {
        name: '按量使用',
        badge: '',
        description: '适合个人工具、测试验证和轻量 AI 编程调用，无需固定包月。',
        price: '按量',
        unit: '用多少算多少',
        features: ['按实际用量计费', '不限时间使用', '支持 AI 工具接入', '用量看板'],
        cta: '开始按量使用',
        featured: false
      },
      {
        name: '高频使用',
        badge: '推荐',
        description: '适合持续开发、团队工具和自动化任务，让 AI 工具全天候按需调用。',
        price: '灵活',
        unit: '适合持续 AI 工作流',
        features: ['高频工具使用', '团队友好接入', '自动化任务支持', '统一用量管理'],
        cta: '开启高频使用',
        featured: true
      }
    ],
    faqEyebrow: 'FAQ',
    faqTitle: '接入前常见问题',
    faqDescription: '核心答案很简单：无需包月、不限时间，主流 AI 工具可通过 API Key 接入。',
    finalCta: '立即获取 API Key',
    faqs: [
      { question: '是否需要包月？', answer: '不需要，按实际用量计费。' },
      { question: 'Token怎么定价？', answer: '按各家Plus最大用量折合Token定价' },
      { question: '支持哪些 AI 工具？', answer: '支持 Codex、Claude Code、openclaw、Hermes 等主流 AI 工具接入场景。' },
      { question: '如何开始使用？', answer: '注册后获取 API Key，并按工具要求配置 API 地址和密钥。' }
    ]
  }
})

// Current year for footer
const currentYear = computed(() => new Date().getFullYear())

// Toggle theme
function toggleTheme() {
  isDark.value = !isDark.value
  document.documentElement.classList.toggle('dark', isDark.value)
  localStorage.setItem('theme', isDark.value ? 'dark' : 'light')
}

// Initialize theme
function initTheme() {
  const savedTheme = localStorage.getItem('theme')
  if (savedTheme === 'dark' || (!savedTheme && window.matchMedia('(prefers-color-scheme: dark)').matches)) {
    isDark.value = true
    document.documentElement.classList.add('dark')
  }
}

onMounted(() => {
  initTheme()

  // Check auth state
  authStore.checkAuth()

  // Ensure public settings are loaded (will use cache if already loaded from injected config)
  if (!appStore.publicSettingsLoaded) {
    appStore.fetchPublicSettings()
  }
})
</script>

<style scoped>
.landing-page {
  letter-spacing: 0;
  color: #111827;
  background:
    radial-gradient(circle at top left, rgba(217, 70, 239, 0.18), transparent 32rem),
    radial-gradient(circle at top right, rgba(99, 102, 241, 0.16), transparent 30rem),
    linear-gradient(180deg, #ffffff 0%, #f8fafc 42%, #eef2ff 100%);
}

html.dark .landing-page {
  color: #f8fafc;
  background: #0b0b0f;
}

.landing-grid {
  background-image:
    linear-gradient(rgba(99, 102, 241, 0.07) 1px, transparent 1px),
    linear-gradient(90deg, rgba(217, 70, 239, 0.06) 1px, transparent 1px);
  background-size: 68px 68px;
  mask-image: linear-gradient(to bottom, black 8%, transparent 74%);
}

html.dark .landing-page .landing-grid {
  background-image:
    linear-gradient(rgba(139, 92, 246, 0.08) 1px, transparent 1px),
    linear-gradient(90deg, rgba(217, 70, 239, 0.06) 1px, transparent 1px);
}

.landing-wash {
  background:
    linear-gradient(110deg, rgba(139, 92, 246, 0.1), transparent 34%, rgba(217, 70, 239, 0.08) 68%, transparent),
    radial-gradient(circle at 48% 4%, rgba(255, 255, 255, 0.92), transparent 28rem);
}

html.dark .landing-page .landing-wash {
  background:
    radial-gradient(circle at 50% 0%, rgba(99, 102, 241, 0.2), transparent 38%),
    linear-gradient(135deg, rgba(217, 70, 239, 0.08), transparent 42%, rgba(20, 184, 166, 0.08));
}

.scan-line {
  background: linear-gradient(90deg, transparent, rgba(217, 70, 239, 0.45), transparent);
}

html.dark .landing-page .scan-line {
  background: linear-gradient(90deg, transparent, rgba(217, 70, 239, 0.7), transparent);
}

.brand-mark {
  border: 1px solid rgba(221, 214, 254, 0.95);
  background: rgba(255, 255, 255, 0.88);
  box-shadow: 0 14px 34px rgba(99, 102, 241, 0.14);
}

html.dark .landing-page .brand-mark {
  border-color: rgba(255, 255, 255, 0.1);
  background: rgba(255, 255, 255, 0.1);
  box-shadow: 0 0 24px rgba(217, 70, 239, 0.28);
}

.landing-brand,
.hero-title,
.landing-strong {
  color: #111827;
}

.hero-title {
  line-height: 1.3 !important;
}

html.dark .landing-page .landing-brand,
html.dark .landing-page .hero-title,
html.dark .landing-page .landing-strong {
  color: #ffffff;
}

.hero-title-gradient {
  background-image: linear-gradient(90deg, #8b5cf6, #d946ef, #6366f1);
}

html.dark .landing-page .hero-title-gradient {
  background-image: linear-gradient(90deg, #f0abfc, #c4b5fd, #a5f3fc);
}

.landing-body {
  color: #334155;
}

.landing-muted {
  color: #64748b;
}

.landing-subtle {
  color: #64748b;
}

html.dark .landing-page .landing-body {
  color: #cbd5e1;
}

html.dark .landing-page .landing-muted {
  color: #94a3b8;
}

html.dark .landing-page .landing-subtle {
  color: #94a3b8;
}

.hero-badge {
  color: #7c3aed;
  border: 1px solid #ddd6fe;
  background: rgba(255, 255, 255, 0.86);
  box-shadow: 0 16px 42px rgba(99, 102, 241, 0.12);
}

html.dark .landing-page .hero-badge {
  color: #fae8ff;
  border-color: rgba(240, 171, 252, 0.2);
  background: rgba(255, 255, 255, 0.06);
  box-shadow: 0 0 28px rgba(217, 70, 239, 0.18);
}

.metric-card {
  border: 1px solid #e5e7eb;
  background: rgba(255, 255, 255, 0.82);
  box-shadow: 0 16px 42px rgba(15, 23, 42, 0.07);
}

html.dark .landing-page .metric-card {
  border-color: rgba(255, 255, 255, 0.1);
  background: rgba(255, 255, 255, 0.05);
  box-shadow: none;
}

.advantage-icon {
  box-shadow: inset 0 0 24px rgba(255, 255, 255, 0.54);
}

.advantage-icon-fuchsia {
  color: #d946ef;
  background: rgba(217, 70, 239, 0.11);
}

.advantage-icon-emerald {
  color: #059669;
  background: rgba(16, 185, 129, 0.11);
}

.advantage-icon-cyan {
  color: #0891b2;
  background: rgba(6, 182, 212, 0.11);
}

.advantage-icon-amber {
  color: #d97706;
  background: rgba(245, 158, 11, 0.13);
}

html.dark .landing-page .advantage-icon {
  box-shadow: none;
}

html.dark .landing-page .advantage-icon-fuchsia {
  color: #f5d0fe;
  background: rgba(217, 70, 239, 0.15);
}

html.dark .landing-page .advantage-icon-emerald {
  color: #a7f3d0;
  background: rgba(52, 211, 153, 0.15);
}

html.dark .landing-page .advantage-icon-cyan {
  color: #a5f3fc;
  background: rgba(34, 211, 238, 0.15);
}

html.dark .landing-page .advantage-icon-amber {
  color: #fde68a;
  background: rgba(251, 191, 36, 0.15);
}

.landing-icon-button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 2.25rem;
  height: 2.25rem;
  border-radius: 9999px;
  color: #475569;
  background: rgba(255, 255, 255, 0.86);
  border: 1px solid #e5e7eb;
  box-shadow: 0 10px 26px rgba(15, 23, 42, 0.06);
  transition:
    color 180ms ease,
    border-color 180ms ease,
    background 180ms ease;
}

.landing-icon-button:hover {
  color: #8b5cf6;
  background: #ffffff;
  border-color: #ddd6fe;
}

html.dark .landing-page .landing-icon-button {
  color: rgb(203 213 225);
  background: rgba(255, 255, 255, 0.06);
  border-color: rgba(255, 255, 255, 0.1);
  box-shadow: none;
}

html.dark .landing-page .landing-icon-button:hover {
  color: white;
  background: rgba(255, 255, 255, 0.12);
  border-color: rgba(217, 70, 239, 0.35);
}

.login-link {
  color: #475569;
}

.login-link:hover {
  color: #8b5cf6;
  background: rgba(255, 255, 255, 0.82);
}

html.dark .landing-page .login-link {
  color: #cbd5e1;
}

html.dark .landing-page .login-link:hover {
  color: white;
  background: rgba(255, 255, 255, 0.1);
}

.register-pill {
  color: white;
  background: linear-gradient(135deg, #8b5cf6, #d946ef);
  box-shadow: 0 12px 32px rgba(139, 92, 246, 0.28);
}

.register-pill:hover {
  box-shadow: 0 14px 36px rgba(217, 70, 239, 0.28);
}

html.dark .landing-page .register-pill {
  color: #0b0b0f;
  background: white;
  box-shadow: 0 0 24px rgba(255, 255, 255, 0.18);
}

html.dark .landing-page .register-pill:hover {
  background: #fae8ff;
}

.dashboard-pill {
  color: #111827;
  border: 1px solid #e5e7eb;
  background: rgba(255, 255, 255, 0.86);
  box-shadow: 0 12px 30px rgba(15, 23, 42, 0.06);
}

.dashboard-pill:hover {
  color: #8b5cf6;
  border-color: #ddd6fe;
  background: #ffffff;
}

html.dark .landing-page .dashboard-pill {
  color: white;
  border-color: rgba(255, 255, 255, 0.1);
  background: rgba(255, 255, 255, 0.1);
}

html.dark .landing-page .dashboard-pill:hover {
  border-color: rgba(240, 171, 252, 0.4);
  background: rgba(255, 255, 255, 0.15);
}

.landing-primary-button,
.landing-secondary-button {
  display: inline-flex;
  min-height: 3rem;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  border-radius: 9999px;
  padding: 0.75rem 1.25rem;
  font-size: 0.9375rem;
  font-weight: 800;
  transition:
    transform 180ms ease,
    box-shadow 180ms ease,
    background 180ms ease,
    border-color 180ms ease;
}

.landing-primary-button {
  color: white;
  background: linear-gradient(135deg, #8b5cf6, #d946ef);
  box-shadow: 0 12px 32px rgba(139, 92, 246, 0.28);
}

.landing-primary-button:hover,
.landing-secondary-button:hover {
  transform: translateY(-1px);
}

.landing-primary-button:hover {
  box-shadow: 0 18px 46px rgba(217, 70, 239, 0.3);
}

.landing-secondary-button {
  color: #8b5cf6;
  background: #ffffff;
  border: 1px solid #ddd6fe;
  box-shadow: 0 12px 30px rgba(99, 102, 241, 0.08);
}

.landing-secondary-button:hover {
  background: white;
  border-color: rgba(217, 70, 239, 0.42);
  box-shadow: 0 16px 38px rgba(139, 92, 246, 0.14);
}

html.dark .landing-page .landing-primary-button {
  box-shadow: 0 0 34px rgba(217, 70, 239, 0.38);
}

html.dark .landing-page .landing-primary-button:hover {
  box-shadow: 0 0 44px rgba(217, 70, 239, 0.5);
}

html.dark .landing-page .landing-secondary-button {
  color: rgb(248 250 252);
  background: rgba(255, 255, 255, 0.06);
  border-color: rgba(255, 255, 255, 0.14);
  box-shadow: none;
}

html.dark .landing-page .landing-secondary-button:hover {
  background: rgba(255, 255, 255, 0.1);
  border-color: rgba(103, 232, 249, 0.34);
}

.api-visual,
.code-showcase,
.landing-card,
.pricing-card,
.faq-item,
.scenario-row {
  border: 1px solid #e5e7eb;
  background:
    linear-gradient(180deg, rgba(255, 255, 255, 0.96), rgba(255, 255, 255, 0.84)),
    rgba(255, 255, 255, 0.86);
  backdrop-filter: blur(16px);
  box-shadow: 0 20px 60px rgba(15, 23, 42, 0.08);
}

html.dark .landing-page .api-visual,
html.dark .landing-page .code-showcase,
html.dark .landing-page .landing-card,
html.dark .landing-page .pricing-card,
html.dark .landing-page .faq-item,
html.dark .landing-page .scenario-row {
  border-color: rgba(255, 255, 255, 0.1);
  background:
    linear-gradient(180deg, rgba(255, 255, 255, 0.085), rgba(255, 255, 255, 0.035)),
    rgba(11, 11, 15, 0.68);
  box-shadow: none;
}

.api-visual {
  overflow: hidden;
  border-radius: 1.5rem;
  box-shadow:
    0 28px 80px rgba(15, 23, 42, 0.12),
    0 0 64px rgba(139, 92, 246, 0.16);
}

html.dark .landing-page .api-visual {
  box-shadow:
    0 22px 70px rgba(0, 0, 0, 0.45),
    0 0 48px rgba(139, 92, 246, 0.24);
}

.api-visual-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  border-bottom: 1px solid #e5e7eb;
  padding: 0.9rem 1rem;
  color: #64748b;
  font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, monospace;
  font-size: 0.75rem;
}

html.dark .landing-page .api-visual-header {
  border-bottom-color: rgba(255, 255, 255, 0.1);
  color: rgb(148 163 184);
}

.api-route-map {
  display: grid;
  grid-template-columns: minmax(0, 1fr) 2rem minmax(0, 1fr) 2rem minmax(0, 1fr);
  align-items: center;
  gap: 0.75rem;
  padding: 1.25rem;
}

.route-node {
  display: grid;
  min-height: 7.5rem;
  place-items: center;
  gap: 0.65rem;
  border-radius: 0.75rem;
  border: 1px solid #e5e7eb;
  background: rgba(255, 255, 255, 0.86);
  text-align: center;
  color: #182033;
  font-size: 0.8125rem;
  font-weight: 700;
}

.route-node-user {
  color: #6366f1;
  box-shadow: inset 0 0 30px rgba(99, 102, 241, 0.08);
}

.route-node-gateway {
  color: #d946ef;
  box-shadow: inset 0 0 34px rgba(217, 70, 239, 0.1);
}

html.dark .landing-page .route-node {
  border-color: rgba(255, 255, 255, 0.1);
  background: rgba(255, 255, 255, 0.055);
  color: white;
}

html.dark .landing-page .route-node-user {
  color: rgb(103 232 249);
}

html.dark .landing-page .route-node-gateway {
  color: rgb(240 171 252);
  box-shadow: inset 0 0 28px rgba(217, 70, 239, 0.12);
}

.route-line {
  height: 2px;
  overflow: hidden;
  border-radius: 999px;
  background: linear-gradient(90deg, transparent, rgba(217, 70, 239, 0.9), transparent);
  position: relative;
}

.route-line::after {
  content: '';
  position: absolute;
  inset: 0;
  background: linear-gradient(90deg, transparent, white, transparent);
  animation: route-pulse 2.2s linear infinite;
}

.provider-chip {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  min-height: 2.15rem;
  border-radius: 9999px;
  border: 1px solid #e5e7eb;
  background: rgba(255, 255, 255, 0.9);
  padding: 0.45rem 0.65rem;
  color: #334155;
  font-size: 0.78rem;
  font-weight: 700;
}

html.dark .landing-page .provider-chip {
  border-color: rgba(255, 255, 255, 0.1);
  background: rgba(255, 255, 255, 0.055);
  color: rgb(226 232 240);
}

.code-panel,
.code-showcase {
  overflow: hidden;
  background:
    linear-gradient(135deg, rgba(15, 23, 42, 0.98), rgba(49, 46, 129, 0.94)),
    #0f172a;
}

.code-panel-header,
.code-showcase-header {
  color: #94a3b8;
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
}

.code-panel {
  margin: 0 1.25rem 1.25rem;
  border-radius: 1rem;
  border: 1px solid rgba(255, 255, 255, 0.1);
}

.code-panel pre,
.code-showcase pre {
  margin: 0;
  overflow-x: auto;
  padding: 1rem;
  color: rgb(226 232 240);
  font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, monospace;
  font-size: 0.8125rem;
  line-height: 1.8;
}

.api-activity {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  border-top: 1px solid #e5e7eb;
}

.activity-row {
  display: grid;
  gap: 0.3rem;
  padding: 1rem;
  color: #64748b;
  font-size: 0.75rem;
}

.activity-row + .activity-row {
  border-left: 1px solid #e5e7eb;
}

.activity-row strong {
  color: #111827;
  font-size: 1rem;
}

html.dark .landing-page .api-activity {
  border-top-color: rgba(255, 255, 255, 0.1);
}

html.dark .landing-page .activity-row {
  color: rgb(148 163 184);
}

html.dark .landing-page .activity-row + .activity-row {
  border-left-color: rgba(255, 255, 255, 0.1);
}

html.dark .landing-page .activity-row strong {
  color: white;
}

.section-heading {
  margin-left: auto;
  margin-right: auto;
  max-width: 46rem;
  text-align: center;
}

.section-heading.text-left {
  margin-left: 0;
  margin-right: 0;
  text-align: left;
}

.section-heading p {
  color: #8b5cf6;
  font-size: 0.75rem;
  font-weight: 800;
  text-transform: uppercase;
}

html.dark .landing-page .section-heading p {
  color: rgb(240 171 252);
}

.section-heading h2 {
  margin-top: 0.75rem;
  color: #111827;
  font-size: clamp(2rem, 4vw, 3.25rem);
  font-weight: 900;
  line-height: 1.12;
}

html.dark .landing-page .section-heading h2 {
  color: white;
}

.section-heading span {
  display: block;
  margin-top: 1rem;
  color: #64748b;
  font-size: 1rem;
  line-height: 1.8;
}

html.dark .landing-page .section-heading span {
  color: rgb(148 163 184);
}

.landing-card,
.scenario-row,
.pricing-card {
  border-radius: 1.5rem;
}

.landing-card {
  padding: 1.25rem;
}

.landing-card:hover,
.scenario-row:hover,
.pricing-card:hover {
  border-color: #ddd6fe;
  box-shadow: 0 24px 70px rgba(139, 92, 246, 0.13);
}

html.dark .landing-page .landing-card:hover,
html.dark .landing-page .scenario-row:hover,
html.dark .landing-page .pricing-card:hover {
  border-color: rgba(217, 70, 239, 0.35);
  box-shadow: none;
}

.scenario-row {
  display: flex;
  min-height: 7.25rem;
  gap: 1rem;
  padding: 1rem;
}

.code-showcase {
  border-radius: 1.5rem;
  box-shadow:
    0 24px 70px rgba(15, 23, 42, 0.18),
    0 0 52px rgba(139, 92, 246, 0.14);
}

.landing-band {
  border-top: 1px solid rgba(229, 231, 235, 0.86);
  border-bottom: 1px solid rgba(229, 231, 235, 0.86);
  background:
    linear-gradient(135deg, rgba(255, 255, 255, 0.7), rgba(238, 242, 255, 0.62)),
    rgba(255, 255, 255, 0.5);
}

html.dark .landing-page .landing-band {
  border-color: rgba(255, 255, 255, 0.1);
  background: rgba(255, 255, 255, 0.025);
}

.integration-band {
  background: linear-gradient(180deg, rgba(255, 255, 255, 0.74), rgba(248, 250, 252, 0));
}

html.dark .landing-page .integration-band {
  background: linear-gradient(180deg, rgba(255, 255, 255, 0.04), transparent);
}

.step-row {
  border: 1px solid #e5e7eb;
  background: rgba(255, 255, 255, 0.86);
  box-shadow: 0 16px 42px rgba(15, 23, 42, 0.06);
}

html.dark .landing-page .step-row {
  border-color: rgba(255, 255, 255, 0.1);
  background: rgba(255, 255, 255, 0.05);
}

.step-index,
.plan-badge {
  color: #8b5cf6;
  background: rgba(139, 92, 246, 0.1);
}

html.dark .landing-page .step-index,
html.dark .landing-page .plan-badge {
  color: #f5d0fe;
  background: rgba(217, 70, 239, 0.15);
}

.pricing-card {
  padding: 1.5rem;
}

.pricing-card-featured {
  border-color: #ddd6fe;
  background:
    linear-gradient(180deg, rgba(255, 255, 255, 0.96), rgba(250, 245, 255, 0.88)),
    rgba(255, 255, 255, 0.9);
  box-shadow:
    0 24px 76px rgba(139, 92, 246, 0.16),
    0 0 0 1px rgba(217, 70, 239, 0.08);
}

html.dark .landing-page .pricing-card-featured {
  border-color: rgba(217, 70, 239, 0.36);
  box-shadow: 0 0 42px rgba(217, 70, 239, 0.18);
}

.plan-button {
  color: white;
  background: linear-gradient(135deg, #8b5cf6, #d946ef);
  box-shadow: 0 12px 32px rgba(139, 92, 246, 0.22);
}

.plan-button:hover {
  box-shadow: 0 16px 42px rgba(217, 70, 239, 0.26);
}

html.dark .landing-page .plan-button {
  color: #0b0b0f;
  background: white;
  box-shadow: none;
}

html.dark .landing-page .plan-button:hover {
  background: #fae8ff;
}

.faq-item {
  border-radius: 1.25rem;
  padding: 1rem 1.125rem;
}

.faq-item summary {
  cursor: pointer;
  color: #111827;
  font-weight: 800;
  list-style: none;
}

html.dark .landing-page .faq-item summary {
  color: white;
}

.faq-item summary::-webkit-details-marker {
  display: none;
}

.faq-item summary::after {
  content: '+';
  float: right;
  color: #d946ef;
}

.faq-item[open] summary::after {
  content: '-';
}

.faq-item p {
  margin-top: 0.75rem;
  color: #64748b;
  font-size: 0.925rem;
  line-height: 1.75;
}

html.dark .landing-page .faq-item p {
  color: rgb(148 163 184);
}

.faq-section,
.landing-footer {
  border-top: 1px solid rgba(229, 231, 235, 0.9);
}

html.dark .landing-page .faq-section,
html.dark .landing-page .landing-footer {
  border-top-color: rgba(255, 255, 255, 0.1);
}

.footer-text,
.footer-link {
  color: #64748b;
}

.footer-link:hover {
  color: #111827;
}

html.dark .landing-page .footer-text,
html.dark .landing-page .footer-link {
  color: #64748b;
}

html.dark .landing-page .footer-link:hover {
  color: white;
}

.scan-line {
  animation: scan 8s linear infinite;
}

@keyframes scan {
  0% {
    transform: translateY(0);
    opacity: 0;
  }
  10%,
  80% {
    opacity: 1;
  }
  100% {
    transform: translateY(100vh);
    opacity: 0;
  }
}

@keyframes route-pulse {
  from {
    transform: translateX(-100%);
  }
  to {
    transform: translateX(100%);
  }
}

@media (max-width: 640px) {
  .api-route-map {
    grid-template-columns: 1fr;
  }

  .route-line {
    height: 2rem;
    width: 2px;
    justify-self: center;
    background: linear-gradient(180deg, transparent, rgba(217, 70, 239, 0.9), transparent);
  }

  .api-activity {
    grid-template-columns: 1fr;
  }

  .activity-row + .activity-row {
    border-left: 0;
    border-top: 1px solid rgba(255, 255, 255, 0.1);
  }

  .landing-page .activity-row + .activity-row {
    border-top-color: #e5e7eb;
  }
}

@media (prefers-reduced-motion: reduce) {
  .scan-line,
  .route-line::after {
    animation: none;
  }

  .landing-primary-button,
  .landing-secondary-button {
    transition: none;
  }
}
</style>
