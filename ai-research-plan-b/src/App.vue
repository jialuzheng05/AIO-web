<script setup lang="ts">
import { computed, onBeforeUnmount, onMounted, ref } from 'vue'
import citeIcon from './assets/cite.svg'
import collectIcon from './assets/collect.svg'
import historyIcon from './assets/history_2.svg'
import listArrowIcon from './assets/list_arrow.svg'

type NavItem = {
  label: string
  icon: string
  active?: boolean
}

type Source = {
  title: string
  quotes: string
  quoteDetails: Array<{
    label: string
    lead: string
    body: string
  }>
  takeaway: string
  year: string
  author: string
  type: string
  citationCount?: string
}

const navItems: NavItem[] = [
  { label: 'Home', icon: 'home' },
  { label: 'AI Study', icon: 'book' },
  { label: 'AI Live Notes', icon: 'mic' },
  { label: 'AI Writing Tools', icon: 'spark', active: true },
  { label: 'Exam Predictor', icon: 'exam' },
  { label: 'Mini Games', icon: 'game' },
  { label: 'Apps', icon: 'grid' },
]

const tabs = [
  'AI Humanizer',
  'AI Detector',
  'Plagiarism Checker',
  'Paraphraser',
  'Citation Machine',
  'AI Research',
]

const searchQuery = 'Educational data science in higher education'

const projects = ['AI Research 方案B'] as const
type ProjectName = (typeof projects)[number]
const selectedProject = ref<ProjectName>('AI Research 方案B')
const isProjectMenuOpen = ref(false)
const projectMenuRef = ref<HTMLElement | null>(null)

const sortOptions = ['Most Relevant', 'Most Cited', 'Newest', 'Saved Sources'] as const
type SortOption = (typeof sortOptions)[number]
const selectedSort = ref<SortOption>('Most Relevant')
const isSortMenuOpen = ref(false)
const sortMenuRef = ref<HTMLElement | null>(null)
const isFilterMenuOpen = ref(false)
const filterMenuRef = ref<HTMLElement | null>(null)

const resourceTypes = ['All', 'Research article', 'Systematic review', 'Book', 'Journal'] as const
type ResourceType = (typeof resourceTypes)[number]
const selectedResourceType = ref<ResourceType>('All')
const isFilterActive = computed(() => selectedResourceType.value !== 'All')

const citationFormats = ['APA', 'MLA', 'Chicago', 'Harvard', 'BibTeX', 'AMA/Numeric'] as const
type CitationFormat = (typeof citationFormats)[number]
const selectedCitationFormat = ref<CitationFormat>('APA')
const activeCitationSource = ref<Source | null>(null)

const toggleProjectMenu = () => {
  isProjectMenuOpen.value = !isProjectMenuOpen.value
  isSortMenuOpen.value = false
  isFilterMenuOpen.value = false
}

const selectProject = (project: ProjectName) => {
  selectedProject.value = project
  isProjectMenuOpen.value = false
}

const toggleSortMenu = () => {
  isSortMenuOpen.value = !isSortMenuOpen.value
  isFilterMenuOpen.value = false
  isProjectMenuOpen.value = false
}

const toggleFilterMenu = () => {
  isFilterMenuOpen.value = !isFilterMenuOpen.value
  isSortMenuOpen.value = false
  isProjectMenuOpen.value = false
}

const selectSortOption = (option: SortOption) => {
  selectedSort.value = option
  isSortMenuOpen.value = false
}

const resetFilters = () => {
  if (!isFilterActive.value) return

  selectedResourceType.value = 'All'
}

const openCitationDialog = (source: Source) => {
  activeCitationSource.value = source
  selectedCitationFormat.value = 'APA'
  isProjectMenuOpen.value = false
  isSortMenuOpen.value = false
  isFilterMenuOpen.value = false
}

const closeCitationDialog = () => {
  activeCitationSource.value = null
}

const getCitationText = (source: Source, format: CitationFormat) => {
  const doi = 'doi:10.48550/arxiv.2508.06729'
  const base = `${source.author}. ${source.title}. ${source.type}. ${source.year}; ${doi}`

  if (format === 'APA') {
    return `${source.author}. (${source.year}). ${source.title}. ${source.type}. ${doi}`
  }

  if (format === 'MLA') {
    return `${source.author}. "${source.title}." ${source.type}, ${source.year}, ${doi}.`
  }

  if (format === 'Chicago') {
    return `${source.author}. "${source.title}." ${source.type} (${source.year}). ${doi}.`
  }

  if (format === 'Harvard') {
    return `${source.author} (${source.year}) '${source.title}', ${source.type}. Available at: ${doi}.`
  }

  if (format === 'BibTeX') {
    return `@article{${source.author.split(' ')[0].toLowerCase()}${source.year},\n  title={${source.title}},\n  author={${source.author}},\n  year={${source.year}},\n  journal={${source.type}},\n  doi={10.48550/arxiv.2508.06729}\n}`
  }

  return base
}

const copyCitationText = async () => {
  if (!activeCitationSource.value) return

  await navigator.clipboard?.writeText(
    getCitationText(activeCitationSource.value, selectedCitationFormat.value),
  )
}

const handleDocumentClick = (event: MouseEvent) => {
  if (!projectMenuRef.value?.contains(event.target as Node)) {
    isProjectMenuOpen.value = false
  }

  if (!sortMenuRef.value?.contains(event.target as Node)) {
    isSortMenuOpen.value = false
  }

  if (!filterMenuRef.value?.contains(event.target as Node)) {
    isFilterMenuOpen.value = false
  }
}

onMounted(() => {
  document.addEventListener('click', handleDocumentClick)
})

onBeforeUnmount(() => {
  document.removeEventListener('click', handleDocumentClick)
})

const sources: Source[] = [
  {
    title: 'Teaching computational archival science: context, pedagogy, and future directions',
    quotes: '3 Relevant Quote',
    quoteDetails: [
      {
        label: 'QUOTE 1',
        lead: 'Strong experimental evidence linking',
        body: 'loss to inflammatory markers-ideal for mechanism-focused arguments in your paper focused arguments in your paper.',
      },
      {
        label: 'QUOTE 2',
        lead: 'Strong experimental evidence linking',
        body: 'loss to inflammatory markers-ideal for mechanism-focused arguments in your paper focused arguments in your paper.',
      },
      {
        label: 'QUOTE 3',
        lead: 'Strong experimental evidence linking',
        body: 'loss to inflammatory markers-ideal for mechanism-focused arguments in your paper focused arguments in your paper.',
      },
    ],
    takeaway:
      'Strong experimental evidence linking acute sleep loss to inflammatory markers-ideal for mechanism-focused arguments in your paper.',
    year: '2001',
    author: 'Shearer et al.',
    type: 'Research Article',
    citationCount: '1,240',
  },
  {
    title: 'Educational data mining and learning analytics in higher education',
    quotes: '2 Relevant Quote',
    quoteDetails: [
      {
        label: 'QUOTE 1',
        lead: 'Useful framing for institutional data',
        body: 'can reveal learning patterns and improve student support decisions.',
      },
      {
        label: 'QUOTE 2',
        lead: 'Learning analytics interventions',
        body: 'are strongest when paired with advising workflows and transparent student support.',
      },
    ],
    takeaway:
      'Useful framing for how institutional data can reveal learning patterns and improve student support decisions.',
    year: '2016',
    author: 'Siemens et al.',
    type: 'Journal Paper',
    citationCount: '986',
  },
  {
    title: 'A systematic review of predictive analytics for student success',
    quotes: '4 Relevant Quote',
    quoteDetails: [
      {
        label: 'QUOTE 1',
        lead: 'Early-warning models',
        body: 'work best when paired with transparent interventions and advising workflows.',
      },
      {
        label: 'QUOTE 2',
        lead: 'Predictive systems',
        body: 'need clear accountability and consistent follow-up to improve outcomes.',
      },
      {
        label: 'QUOTE 3',
        lead: 'Student success indicators',
        body: 'combine academic activity, engagement signals, and contextual support data.',
      },
    ],
    takeaway:
      'Clear evidence that early-warning models work best when paired with transparent interventions and advising workflows.',
    year: '2019',
    author: 'Ifenthaler et al.',
    type: 'Review Article',
    citationCount: '742',
  },
  {
    title: 'Ethical challenges of educational data science in universities',
    quotes: '1 Relevant Quote',
    quoteDetails: [
      {
        label: 'QUOTE 1',
        lead: 'Privacy and consent',
        body: 'remain central risks when universities operationalize student data science.',
      },
    ],
    takeaway:
      'Strong source for privacy, consent, and bias concerns when using student data in higher education systems.',
    year: '2020',
    author: 'Prinsloo et al.',
    type: 'Conference Paper',
    citationCount: '518',
  },
  {
    title: 'Data-informed teaching practice and student engagement outcomes',
    quotes: '2 Relevant Quote',
    quoteDetails: [
      {
        label: 'QUOTE 1',
        lead: 'Instructor-facing dashboards',
        body: 'can translate learning signals into practical classroom decisions.',
      },
      {
        label: 'QUOTE 2',
        lead: 'Data-informed practice',
        body: 'improves engagement when teachers can connect indicators to interventions.',
      },
    ],
    takeaway:
      'Connects analytics dashboards to classroom decision-making, with practical examples for instructor-facing tools.',
    year: '2022',
    author: 'Wise et al.',
    type: 'Research Article',
    citationCount: '365',
  },
]
</script>

<template>
  <div class="app-frame">
    <header class="control-panel">
      <div class="project-nav">
        <span class="control-label">项目导航</span>
        <div ref="projectMenuRef" class="project-switcher">
          <button
            class="project-trigger"
            type="button"
            :aria-expanded="isProjectMenuOpen"
            aria-haspopup="menu"
            @click.stop="toggleProjectMenu"
            @keydown.esc="isProjectMenuOpen = false"
          >
            <span>{{ selectedProject }}</span>
            <svg viewBox="0 0 20 20" aria-hidden="true">
              <path d="m6.5 8.2 3.5 3.5 3.5-3.5" />
            </svg>
          </button>
          <div v-if="isProjectMenuOpen" class="project-menu" role="menu">
            <button
              v-for="project in projects"
              :key="project"
              type="button"
              role="menuitemradio"
              :aria-checked="selectedProject === project"
              :class="{ active: selectedProject === project }"
              @click.stop="selectProject(project)"
            >
              {{ project }}
            </button>
          </div>
        </div>
      </div>

      <div class="control-meta">
        <span>页面版本: <strong>V2</strong></span>
        <span>页面状态: <strong>正常态</strong></span>
        <span>状态版本: <strong>S1(默认)</strong></span>
      </div>
    </header>

    <main class="app-shell">
    <aside class="sidebar">
      <div class="brand-row">
        <div class="brand-mark" aria-hidden="true">
          <svg viewBox="0 0 24 24">
            <path d="M7.4 4.5h9.2c1.6 0 2.9 1.3 2.9 2.9v9.2c0 1.6-1.3 2.9-2.9 2.9H7.4a2.9 2.9 0 0 1-2.9-2.9V7.4c0-1.6 1.3-2.9 2.9-2.9Z" />
            <path d="M8.3 12c1.4-2.4 2.8-3.6 4.1-3.6 1.1 0 1.9.6 2.7 1.7m.6 1.9c-1.4 2.4-2.8 3.6-4.1 3.6-1.1 0-1.9-.6-2.7-1.7" />
            <path d="m7.1 8.8 2.2 2.1m5.4 2.2 2.2 2.1m0-6.4-2.2 2.1m-5.4 2.2-2.2 2.1" />
          </svg>
        </div>
        <strong>Solvely.ai</strong>
        <button class="collapse-button" aria-label="Collapse sidebar">
          <svg viewBox="0 0 20 20">
            <path d="M5.5 4.8h9v10.4h-9z" />
            <path d="M9 5v10M12.8 8.2 11 10l1.8 1.8" />
          </svg>
        </button>
      </div>

      <nav class="nav-list" aria-label="Primary">
        <a v-for="item in navItems" :key="item.label" class="nav-item" :class="{ active: item.active }" href="#">
          <span class="nav-icon" :data-icon="item.icon" aria-hidden="true">
            <svg v-if="item.icon === 'home'" viewBox="0 0 20 20"><path d="M3.5 9.4 10 4l6.5 5.4M5.8 8.2v7.1h8.4V8.2M8.7 15.3v-4h2.6v4" /></svg>
            <svg v-else-if="item.icon === 'book'" viewBox="0 0 20 20"><path d="M4.3 5.2h4.2c.8 0 1.5.6 1.5 1.4v9.1c0-.8-.7-1.4-1.5-1.4H4.3zM10 6.6c0-.8.7-1.4 1.5-1.4h4.2v9.1h-4.2c-.8 0-1.5.6-1.5 1.4" /><path d="M6.2 7.8h1.9M11.9 7.8h1.9" /></svg>
            <svg v-else-if="item.icon === 'mic'" viewBox="0 0 20 20"><path d="M10 3.8a2.3 2.3 0 0 1 2.3 2.3V10a2.3 2.3 0 0 1-4.6 0V6.1A2.3 2.3 0 0 1 10 3.8Z" /><path d="M5.8 9.6a4.2 4.2 0 0 0 8.4 0M10 13.8v2.4M7.6 16.2h4.8" /></svg>
            <svg v-else-if="item.icon === 'spark'" viewBox="0 0 20 20"><path d="m10 3.5 1.1 3.8 3.6 1.4-3.6 1.4L10 14l-1.1-3.9-3.6-1.4 3.6-1.4zM15.1 12.5l.5 1.5 1.4.5-1.4.5-.5 1.5-.5-1.5-1.4-.5 1.4-.5z" /></svg>
            <svg v-else-if="item.icon === 'exam'" viewBox="0 0 20 20"><path d="M5.4 3.8h9.2c.8 0 1.4.6 1.4 1.4v9.6c0 .8-.6 1.4-1.4 1.4H5.4c-.8 0-1.4-.6-1.4-1.4V5.2c0-.8.6-1.4 1.4-1.4Z" /><path d="M7 7h6M7 10h6M7 13h3" /></svg>
            <svg v-else-if="item.icon === 'game'" viewBox="0 0 20 20"><path d="M4.8 7.1h10.4c.9 0 1.6.7 1.6 1.6v4.6c0 .9-.7 1.6-1.6 1.6H4.8c-.9 0-1.6-.7-1.6-1.6V8.7c0-.9.7-1.6 1.6-1.6Z" /><path d="M6.9 9.3v3.4M5.2 11h3.4M13.7 9.9h.1M15.1 12.1h.1" /></svg>
            <svg v-else viewBox="0 0 20 20"><path d="M4.8 4.8h3.2v3.2H4.8zM12 4.8h3.2v3.2H12zM4.8 12h3.2v3.2H4.8zM12 12h3.2v3.2H12z" /></svg>
          </span>
          <span>{{ item.label }}</span>
        </a>
      </nav>

      <div class="sidebar-spacer"></div>

      <a class="pro-link" href="#">
        <span class="pro-badge">x</span>
        <span>All in One Pro</span>
      </a>
    </aside>

    <section class="content-area">
      <div class="watermark" aria-hidden="true"></div>

      <header class="hero">
        <h1>AI Writing Tools</h1>
        <p>
          <span class="sparkle" aria-hidden="true">
            <svg viewBox="0 0 20 20"><path d="m10 3.8 1.1 3.9 3.8 1.4-3.8 1.4-1.1 3.9-1.1-3.9-3.8-1.4 3.8-1.4z" /></svg>
          </span>
          Your All-in-One Writing Assistant
          <span class="sparkle" aria-hidden="true">
            <svg viewBox="0 0 20 20"><path d="m10 3.8 1.1 3.9 3.8 1.4-3.8 1.4-1.1 3.9-1.1-3.9-3.8-1.4 3.8-1.4z" /></svg>
          </span>
        </p>
      </header>

      <section class="workspace-card" aria-label="AI Research">
        <div class="tabs">
          <button v-for="tab in tabs" :key="tab" :class="{ active: tab === 'AI Research' }">{{ tab }}</button>
        </div>

        <div class="workspace-body">
          <section class="search-pane">
            <div class="pane-header">
              <h2>Your Search</h2>
              <div class="header-actions">
                <button>
                  <img class="header-action-icon" :src="collectIcon" alt="" />
                  Saved <span>0</span>
                </button>
                <button>
                  <img class="header-action-icon" :src="historyIcon" alt="" />
                  History
                </button>
              </div>
            </div>

            <textarea aria-label="Search query" spellcheck="false" :value="searchQuery"></textarea>

            <div class="search-footer">
              <span>130 words</span>
              <button class="search-button">
                <svg viewBox="0 0 20 20"><path d="m12.4 5.1 2.5 2.5-7.8 7.8H4.6v-2.5zM10.9 6.6l2.5 2.5M13.3 3.8l2.9 2.9" /><path d="M5.1 6.2h2.2M6.2 5.1v2.2M14 12.7h1.8M14.9 11.8v1.8" /></svg>
                Search
              </button>
            </div>
          </section>

          <section class="results-pane">
            <div class="results-header">
              <h2><span>5</span> Sources Found</h2>
              <div class="result-actions">
                <div ref="sortMenuRef" class="sort-control">
                  <button
                    class="relevance"
                    type="button"
                    :aria-expanded="isSortMenuOpen"
                    aria-haspopup="menu"
                    @click.stop="toggleSortMenu"
                    @keydown.esc="isSortMenuOpen = false"
                  >
                    <img class="list-arrow-icon" :src="listArrowIcon" alt="" />
                    {{ selectedSort }}
                    <svg class="chevron" viewBox="0 0 20 20"><path d="m6.5 8.2 3.5 3.5 3.5-3.5" /></svg>
                  </button>
                  <div v-if="isSortMenuOpen" class="sort-menu" role="menu">
                    <button
                      v-for="option in sortOptions"
                      :key="option"
                      class="sort-menu-item"
                      type="button"
                      role="menuitemradio"
                      :aria-checked="selectedSort === option"
                      @click.stop="selectSortOption(option)"
                    >
                      <span>{{ option }}</span>
                      <svg v-if="selectedSort === option" class="sort-check" viewBox="0 0 24 24" aria-hidden="true">
                        <path d="m5.5 12.5 4.2 4.2 8.8-9.4" />
                      </svg>
                    </button>
                  </div>
                </div>
                <div ref="filterMenuRef" class="filter-control">
                  <button
                    type="button"
                    :class="{ active: isFilterActive }"
                    :aria-expanded="isFilterMenuOpen"
                    aria-haspopup="dialog"
                    @click.stop="toggleFilterMenu"
                    @keydown.esc="isFilterMenuOpen = false"
                  >
                    <svg viewBox="0 0 20 20"><path d="M4 5.2h12M6.8 10h6.4M8.6 14.8h2.8" /><path d="M7 5.2v2.6M13 10v2.6" /></svg>
                    Filters
                  </button>
                  <div v-if="isFilterMenuOpen" class="filter-menu" role="dialog" aria-label="Filters">
                    <section class="filter-section">
                      <h3>Publication Year</h3>
                      <div class="year-range">
                        <button type="button">Start time</button>
                        <span>To</span>
                        <button type="button">End time</button>
                      </div>
                    </section>

                    <section class="filter-section">
                      <h3>Resource Type</h3>
                      <div class="resource-chips">
                        <button
                          v-for="type in resourceTypes"
                          :key="type"
                          type="button"
                          :class="{ active: selectedResourceType === type }"
                          @click="selectedResourceType = type"
                        >
                          {{ type }}
                        </button>
                      </div>
                    </section>

                    <div class="filter-footer">
                      <button
                        class="reset-filter"
                        type="button"
                        :disabled="!isFilterActive"
                        @click="resetFilters"
                      >
                        Reset filter
                      </button>
                    </div>
                  </div>
                </div>
              </div>
            </div>

            <div class="source-list">
              <article v-for="source in sources" :key="`${source.title}-${source.year}`" class="source-card">
                <div class="source-top">
                  <h3>{{ source.title }}</h3>
                  <div class="quote-wrapper">
                    <button class="quote-pill" type="button" aria-haspopup="dialog">
                      <span class="quote-icon" aria-hidden="true"></span>
                      {{ source.quotes }}
                      <svg class="quote-chevron" viewBox="0 0 20 20" aria-hidden="true">
                        <path d="m6.5 8.2 3.5 3.5 3.5-3.5" />
                      </svg>
                    </button>
                    <div class="quote-popover" role="dialog" aria-label="Relevant quotes">
                      <div v-for="quote in source.quoteDetails" :key="`${source.title}-${quote.label}`" class="quote-popover-row">
                        <span class="quote-popover-icon" aria-hidden="true"></span>
                        <p>
                          <strong>{{ quote.label }}</strong>
                          <span>&middot;</span>
                          {{ quote.lead }}
                          <br />
                          {{ quote.body }}
                        </p>
                      </div>
                    </div>
                  </div>
                </div>
                <p class="takeaway"><strong>KEY TAKEWAY</strong> &middot; {{ source.takeaway }}</p>
                <div class="source-meta">
                  <div>
                    <strong>{{ source.year }}</strong>
                    <span>&middot;</span>
                    <span>{{ source.author }}</span>
                    <span>&middot;</span>
                    <span>{{ source.type }}</span>
                  </div>
                  <div class="source-tools">
                    <span v-if="source.citationCount">
                      <strong>{{ source.citationCount }}</strong>
                      Citations
                    </span>
                  </div>
                </div>
                <div class="source-hover-actions">
                  <button class="source-cite-action" type="button" aria-label="Cite source" @click.stop="openCitationDialog(source)">
                    <img :src="citeIcon" alt="" />
                    <span>Cite</span>
                  </button>
                  <button class="source-save-action" type="button" aria-label="Save source">
                    <img :src="collectIcon" alt="" />
                  </button>
                </div>
              </article>
            </div>
          </section>
        </div>
      </section>
    </section>
    </main>

    <div
      v-if="activeCitationSource"
      class="citation-overlay"
      role="presentation"
      @click.self="closeCitationDialog"
    >
      <section class="citation-dialog" role="dialog" aria-modal="true" aria-labelledby="citation-dialog-title">
        <header class="citation-dialog-header">
          <h2 id="citation-dialog-title">Cite this finding</h2>
          <div class="citation-dialog-tools">
            <button type="button" aria-label="Close citation dialog" @click="closeCitationDialog">
              <svg viewBox="0 0 20 20"><path d="M5.5 5.5 14.5 14.5M14.5 5.5 5.5 14.5" /></svg>
            </button>
          </div>
        </header>

        <nav class="citation-tabs" aria-label="Citation formats">
          <button
            v-for="format in citationFormats"
            :key="format"
            type="button"
            :class="{ active: selectedCitationFormat === format }"
            @click="selectedCitationFormat = format"
          >
            {{ format }}
          </button>
        </nav>

        <div class="citation-content">
          <p>{{ getCitationText(activeCitationSource, selectedCitationFormat) }}</p>
        </div>

        <footer class="citation-dialog-footer">
          <button class="citation-copy" type="button" @click="copyCitationText">Copy text</button>
        </footer>
      </section>
    </div>
  </div>
</template>
