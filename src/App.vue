<script setup lang="ts">
import { computed, onBeforeUnmount, onMounted, ref } from 'vue'
import citeIcon from './assets/cite.svg'
import collectIcon from './assets/collect.svg'
import historyIcon from './assets/history_2.svg'
import listArrowIcon from './assets/list_arrow.svg'
import PlanC2Loading from './prototypes/PlanC2Loading.vue'
import PlanC3Loading from './prototypes/PlanC3Loading.vue'
import PlanC4Loading from './prototypes/PlanC4Loading.vue'

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
  relevanceLabel?: string
  relevanceText?: string
  bestUsedFor?: string
  suggestedSentence?: string
  otherWays?: Array<{
    title: string
    sentence: string
  }>
}

type ReferenceCard = {
  title: string
  quoteLead: string
  quoteBody: string
  year: string
  author: string
  type: string
  citationCount: string
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
const planCSuggestions = [
  'Educational data science in higher education',
  'How does sleep paralysis affect mental wellbeing?',
  'Metacognition differences between novices and experts',
]
const planC3PromptCards = [
  {
    kind: 'Topic',
    icon: 'hash',
    text: 'Educational data science in higher education',
  },
  {
    kind: 'Assignment',
    icon: 'notebook',
    text: 'Write a literature review examining the relationship between learning analytics and student success.',
  },
  {
    kind: 'Question',
    icon: 'question',
    text: 'How does sleep paralysis affect mental wellbeing?',
  },
]
const planCGenerationSteps = [
  'Understanding your research question',
  'Searching academic sources',
  'Evaluating credibility and relevance',
  'Preparing your results',
]

const projects = [
  '加侧边栏展开',
  'AI Research 方案C',
  'AI Research 方案C-2',
  'AI Research 方案C-3',
  'AI Research 方案C-4',
  'AI Research 方案C-5',
  'AI research 方案A',
  'AI Research 方案B',
] as const
type ProjectName = (typeof projects)[number]
const projectNavItems = ['加侧边栏展开', 'AI Research'] as const
type ProjectNavName = (typeof projectNavItems)[number]
const debugProjectOptions: Array<{ label: string; project: ProjectName }> = [
  { label: '方案A', project: 'AI research 方案A' },
  { label: '方案C', project: 'AI Research 方案C' },
  { label: '方案C-2', project: 'AI Research 方案C-2' },
  { label: '方案C-3', project: 'AI Research 方案C-3' },
  { label: '方案C-4', project: 'AI Research 方案C-4' },
  { label: '方案C-5', project: 'AI Research 方案C-5' },
]
const selectedProject = ref<ProjectName>(projects[0])
const isPlanBSelected = computed(() => selectedProject.value === 'AI Research 方案B')
const isPlanC2Selected = computed(() => selectedProject.value === 'AI Research 方案C-2')
const isPlanC3Selected = computed(() => selectedProject.value === 'AI Research 方案C-3')
const isPlanC4Selected = computed(() => selectedProject.value === 'AI Research 方案C-4')
const isPlanC5Selected = computed(() => selectedProject.value === 'AI Research 方案C-5')
const isPlanC4LikeSelected = computed(() => isPlanC4Selected.value || isPlanC5Selected.value)
const isPlanCAdvancedSelected = computed(() => isPlanC3Selected.value || isPlanC4LikeSelected.value)
const isPlanCSelected = computed(() =>
  selectedProject.value === 'AI Research 方案C' ||
  selectedProject.value === 'AI Research 方案C-2' ||
  selectedProject.value === 'AI Research 方案C-3' ||
  selectedProject.value === 'AI Research 方案C-4' ||
  selectedProject.value === 'AI Research 方案C-5',
)
const isSidebarExpandProjectSelected = computed(() => selectedProject.value === '加侧边栏展开')
const selectedProjectNav = computed<ProjectNavName>(() =>
  isSidebarExpandProjectSelected.value ? '加侧边栏展开' : 'AI Research',
)
const isDetectorRailExpanded = ref(false)
const planCQuery = ref('')
const hasPlanCQuery = computed(() => planCQuery.value.trim().length > 0)
const hasPlanCResults = ref(false)
const isPlanCGenerating = ref(false)
const isPlanCSearchActive = computed(() => hasPlanCResults.value || isPlanCGenerating.value)
const planCGeneratingStep = ref(0)
const selectedPlanCSourceIndex = ref(0)
const isPlanCSourceLoading = ref(false)
const isPlanCAbstractExpanded = ref(false)
const isPlanC5InlineLoading = ref(false)
const isPlanAAbstractExpanded = ref(false)
const selectedResultView = ref<'Result' | 'Sources'>('Result')
const expandedQuoteSourceKey = ref<string | null>(null)
const expandedPlanASourceKey = ref<string | null>(null)
const selectedPlanASource = ref<Source | null>(null)
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
const pendingResourceType = ref<ResourceType>('All')
const appliedResourceType = ref<ResourceType>('All')
const isFilterActive = computed(() => appliedResourceType.value !== 'All')

const citationFormats = ['APA', 'MLA', 'Chicago', 'Harvard', 'BibTeX', 'AMA/Numeric'] as const
type CitationFormat = (typeof citationFormats)[number]
const selectedCitationFormat = ref<CitationFormat>('APA')
const activeCitationSource = ref<Source | null>(null)
const activeReferenceIndex = ref(0)
const activeReferencePopoverId = ref<string | null>(null)
const referencePopoverStyle = ref<Record<string, string>>({})
let referencePopoverCloseTimer: number | null = null
let planCSearchTimers: number[] = []
let planCSourceLoadingTimer: number | null = null

const referenceCards: ReferenceCard[] = [
  {
    title: 'Teaching computational archival science: context, pedagogy, and future directions',
    quoteLead: 'Strong experimental evidence linking',
    quoteBody:
      'loss to inflammatory markers—ideal for mechanism-focused arguments in your paper focused argument...',
    year: '2001',
    author: 'Shearer et al.',
    type: 'Research Article',
    citationCount: '1,240',
  },
  {
    title: 'Educational data mining and learning analytics in higher education',
    quoteLead: 'Useful framing for how institutional data can reveal',
    quoteBody:
      'learning patterns and improve student support decisions across higher education systems.',
    year: '2016',
    author: 'Siemens et al.',
    type: 'Journal Paper',
    citationCount: '986',
  },
]

const planCResultSources = [
  {
    title: 'Educational data science in higher education: learning analytics, data mining, and institutional decision-making',
    cardTitle: ['Educational data science in higher education'],
    tags: ['Peer-reviewed', 'Open Access', '1,240 Citations'],
    snapshot:
      'Maps how universities use learning analytics, educational data mining, and institutional data to improve teaching, retention, and student support.',
    year: '2021',
    author: 'Ifenthaler et al.',
    venue: 'Computers & Education',
    type: 'Research Paper',
    detailAuthor: 'Ifenthaler ... +6 more',
    abstractIntro:
      'This study reviews how educational data science is used in higher education to interpret learning traces, model student engagement, and support institutional decision-making.',
    abstractMore:
      'It highlights learning management system activity, assessment records, advising data, and course interaction logs as common data sources, while emphasizing that analytics must be paired with ethical governance, transparent interventions, and instructor-facing workflows.',
    usefulSummary:
      'This source is useful for the background and framing section of a paper on educational data science. It explains how learning analytics, data mining, and institutional decision-making connect in higher education, so it can support claims about why student data needs both technical modeling and human intervention workflows.',
    findings: [
      {
        quote:
          'Higher education analytics initiatives often combine LMS activity, assessment data, and advising records to identify student risk and engagement patterns.',
        text: 'Educational data science relies on multiple institutional data streams rather than a single course metric.',
      },
      {
        quote:
          'Analytics systems are most effective when predictions are connected to clear interventions by instructors, advisors, or student support teams.',
        text: 'Predictive models need human follow-up workflows to create measurable student success outcomes.',
      },
    ],
  },
  {
    title: 'Learning analytics dashboards and student success interventions in universities',
    cardTitle: ['Learning analytics dashboards and student success'],
    tags: ['Peer-reviewed', 'Open Access', '986 Citations'],
    snapshot:
      'Shows how dashboards translate course activity and performance signals into actionable support for instructors, advisors, and students.',
    year: '2019',
    author: 'Siemens et al.',
    venue: 'Internet and Higher Education',
    type: 'Journal Article',
    detailAuthor: 'Siemens ... +4 more',
    abstractIntro:
      'This paper examines learning analytics dashboards as a practical layer between educational data science models and day-to-day student support in higher education.',
    abstractMore:
      'The authors describe how visual indicators, risk flags, and engagement summaries can help educators prioritize outreach, but caution that dashboards should explain the meaning of metrics and avoid reducing students to scores.',
    usefulSummary:
      'This source fits well in a section about practical applications of educational data science. It contributes evidence for how dashboards turn raw student activity data into decisions teachers and advisors can act on, making it useful when arguing that analytics tools need to be designed around real intervention moments.',
    findings: [
      {
        quote:
          'Dashboards help instructors interpret student activity data when indicators are tied to meaningful pedagogical actions.',
        text: 'Analytics interfaces are most useful when they translate data into specific teaching or advising decisions.',
      },
      {
        quote:
          'Students and staff benefit from transparent explanations of how engagement metrics are generated and used.',
        text: 'Trust and interpretability are central design concerns for learning analytics tools.',
      },
    ],
  },
  {
    title: 'Predictive models for student retention using educational data science',
    cardTitle: ['Predictive models for student retention'],
    tags: ['Peer-reviewed', 'Method Study', '742 Citations'],
    snapshot:
      'Compares predictive modeling approaches for identifying retention risk across course activity, assessment, and advising datasets.',
    year: '2020',
    author: 'Tempelaar et al.',
    venue: 'Journal of Learning Analytics',
    type: 'Research Article',
    detailAuthor: 'Tempelaar ... +5 more',
    abstractIntro:
      'This article studies how predictive models can support student retention by combining behavioral, demographic, and assessment signals in higher education.',
    abstractMore:
      'It evaluates model interpretability, false positive risks, and the importance of intervention timing when analytics are used by advising teams.',
    usefulSummary:
      'This article is best used in a methods or model-evaluation section. It helps explain how predictive models identify retention risk, what kinds of student signals are useful, and why interpretability matters when analytics results are handed to advisors or student support teams.',
    findings: [
      {
        quote:
          'Retention models are most actionable when risk signals are paired with interpretable features and timely outreach workflows.',
        text: 'Prediction alone is not sufficient; intervention design determines whether analytics improve retention.',
      },
      {
        quote:
          'Combining behavioral traces with assessment outcomes improves early identification of students needing support.',
        text: 'Multi-source data improves the practical value of educational data science models.',
      },
    ],
  },
  {
    title: 'Ethics, privacy, and governance for learning analytics in higher education',
    cardTitle: ['Ethics, privacy, and learning analytics'],
    tags: ['Peer-reviewed', 'Policy Review', '618 Citations'],
    snapshot:
      'Reviews ethical governance practices for using student data in analytics systems, with emphasis on transparency, consent, and bias mitigation.',
    year: '2022',
    author: 'Tsai et al.',
    venue: 'British Journal of Educational Technology',
    type: 'Review Article',
    detailAuthor: 'Tsai ... +3 more',
    abstractIntro:
      'This review examines the ethical and governance challenges that arise when higher education institutions deploy learning analytics and educational data science tools.',
    abstractMore:
      'The authors focus on privacy, student consent, algorithmic bias, explainability, and institutional accountability as necessary conditions for responsible analytics adoption.',
    usefulSummary:
      'This source is useful for the ethics and limitations section of the paper. It contributes arguments about privacy, consent, bias, and accountability, helping show that educational data science should not only optimize predictions but also protect student trust and institutional responsibility.',
    findings: [
      {
        quote:
          'Students are more likely to trust analytics systems when institutions clearly explain what data is collected and how decisions are made.',
        text: 'Transparency is central to ethical educational data science adoption.',
      },
      {
        quote:
          'Governance frameworks should address privacy, bias, and accountability before predictive analytics are scaled across institutions.',
        text: 'Responsible implementation requires policy safeguards, not only technical model accuracy.',
      },
    ],
  },
]

const selectedPlanCSource = computed(() => planCResultSources[selectedPlanCSourceIndex.value])
const getPlanCUsefulFirstSentence = (source: (typeof planCResultSources)[number]) =>
  source.usefulSummary.split('. ')[0]?.replace(/\.$/, '') ?? source.usefulSummary
const shouldUseShortPlanCTitle = computed(() => isPlanC3Selected.value || isPlanC4LikeSelected.value)
const getPlanCDisplayTitle = (source: (typeof planCResultSources)[number]) =>
  shouldUseShortPlanCTitle.value ? source.cardTitle.join(' ') : source.title
const getPlanC4CardTags = (source: (typeof planCResultSources)[number]) => {
  const citationTag = source.tags.find((tag) => tag.includes('Citations'))
  const otherTags = source.tags.filter((tag) => tag !== citationTag)

  return citationTag ? [citationTag, ...otherTags] : source.tags
}
const getPlanC5CardTags = (source: (typeof planCResultSources)[number]) => {
  const citationTag = source.tags.find((tag) => tag.includes('Citations'))
  const otherTags = source.tags.filter((tag) => tag !== citationTag)

  return citationTag ? [...otherTags, citationTag] : source.tags
}
const planCResultCardRefs = ref<HTMLElement[]>([])
const setPlanCResultCardRef = (element: unknown, index: number) => {
  if (element instanceof HTMLElement) planCResultCardRefs.value[index] = element
}
const selectedPlanCSnapshotRows = computed(() => {
  const rows = [
    [
      {
        field: 'Methods',
        value: 'Review of LMS traces, assessment records, advising data, and institutional decision workflows.',
      },
      {
        field: 'Outcomes',
        value: 'Clearer student-risk signals and more actionable support paths for instructors and advisors.',
      },
      {
        field: 'Results',
        value: 'Educational data science improves decisions when analytics connect to transparent interventions.',
      },
    ],
    [
      {
        field: 'Methods',
        value: 'Dashboard evaluation using course activity indicators, engagement summaries, and staff interviews.',
      },
      {
        field: 'Outcomes',
        value: 'Instructors and advisors gained faster visibility into participation gaps and support priorities.',
      },
      {
        field: 'Results',
        value: 'Dashboards were most useful when metrics were easy to interpret and tied to specific actions.',
      },
    ],
    [
      {
        field: 'Methods',
        value: 'Predictive modeling comparison across behavioral traces, assessment results, and advising records.',
      },
      {
        field: 'Outcomes',
        value: 'Earlier detection of retention risk with stronger explanations for why a student was flagged.',
      },
      {
        field: 'Results',
        value: 'Retention models became more useful when paired with timely outreach and interpretable features.',
      },
    ],
    [
      {
        field: 'Methods',
        value: 'Policy review of privacy, consent, explainability, bias, and governance practices in analytics.',
      },
      {
        field: 'Outcomes',
        value: 'A clearer framework for responsible student-data use across institutional analytics programs.',
      },
      {
        field: 'Results',
        value: 'Trust improves when institutions explain data collection, model use, and accountability safeguards.',
      },
    ],
  ]

  return rows[selectedPlanCSourceIndex.value] ?? rows[0]
})

const toggleProjectMenu = () => {
  isProjectMenuOpen.value = !isProjectMenuOpen.value
  isSortMenuOpen.value = false
  isFilterMenuOpen.value = false
}

const selectProject = (project: ProjectName) => {
  selectedProject.value = project
  isProjectMenuOpen.value = false
  selectedPlanASource.value = null

  if (project === 'AI Research 方案B') {
    selectedResultView.value = 'Result'
    expandedPlanASourceKey.value = null
  } else {
    expandedQuoteSourceKey.value = null
    expandedPlanASourceKey.value = null
  }
}

const selectProjectNav = (project: ProjectNavName) => {
  if (project === 'AI Research') {
    selectedProject.value = 'AI Research 方案C-5'
  } else {
    selectedProject.value = project
  }

  isProjectMenuOpen.value = false
  isSortMenuOpen.value = false
  isFilterMenuOpen.value = false
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

  if (isFilterMenuOpen.value) {
    pendingResourceType.value = appliedResourceType.value
  }
}

const selectSortOption = (option: SortOption) => {
  selectedSort.value = option
  isSortMenuOpen.value = false
}

const resetFilters = () => {
  if (!isFilterActive.value) return

  appliedResourceType.value = 'All'
  pendingResourceType.value = 'All'
}

const confirmFilters = () => {
  appliedResourceType.value = pendingResourceType.value
  isFilterMenuOpen.value = false
}

const positionReferencePopover = (event: MouseEvent) => {
  const trigger = event.currentTarget as HTMLElement
  const rect = trigger.getBoundingClientRect()
  const viewportPadding = 16
  const popoverWidth = Math.min(412, window.innerWidth - viewportPadding * 2)
  const estimatedPopoverHeight = 270
  const preferredTop = rect.bottom + 8
  const top =
    preferredTop + estimatedPopoverHeight > window.innerHeight - viewportPadding
      ? Math.max(viewportPadding, rect.top - estimatedPopoverHeight - 8)
      : preferredTop
  const left = Math.min(
    Math.max(viewportPadding, rect.right - popoverWidth),
    window.innerWidth - popoverWidth - viewportPadding,
  )

  referencePopoverStyle.value = {
    top: `${top}px`,
    left: `${left}px`,
    width: `${popoverWidth}px`,
  }
}

const showReferencePopover = (event: MouseEvent, popoverId: string) => {
  if (referencePopoverCloseTimer !== null) {
    window.clearTimeout(referencePopoverCloseTimer)
    referencePopoverCloseTimer = null
  }

  activeReferencePopoverId.value = popoverId
  positionReferencePopover(event)
}

const keepReferencePopoverOpen = () => {
  if (referencePopoverCloseTimer !== null) {
    window.clearTimeout(referencePopoverCloseTimer)
    referencePopoverCloseTimer = null
  }
}

const hideReferencePopoverSoon = () => {
  if (referencePopoverCloseTimer !== null) {
    window.clearTimeout(referencePopoverCloseTimer)
  }

  referencePopoverCloseTimer = window.setTimeout(() => {
    activeReferencePopoverId.value = null
    referencePopoverCloseTimer = null
  }, 240)
}

const showPreviousReference = () => {
  activeReferenceIndex.value =
    activeReferenceIndex.value === 0 ? referenceCards.length - 1 : activeReferenceIndex.value - 1
}

const showNextReference = () => {
  activeReferenceIndex.value = (activeReferenceIndex.value + 1) % referenceCards.length
}

const getSourceKey = (source: Source) => `${source.title}-${source.year}`
const displayedSources = computed(() => {
  if (isPlanBSelected.value) return sources
  if (isPlanCSelected.value) return planCSources

  return planASources
})

const isQuoteExpanded = (source: Source) =>
  isPlanBSelected.value && expandedQuoteSourceKey.value === getSourceKey(source)

const isPlanAExpanded = (source: Source) =>
  !isPlanBSelected.value && expandedPlanASourceKey.value === getSourceKey(source)

const toggleQuotePanel = (source: Source) => {
  if (!isPlanBSelected.value) return

  const sourceKey = getSourceKey(source)
  expandedQuoteSourceKey.value = expandedQuoteSourceKey.value === sourceKey ? null : sourceKey
}

const togglePlanAExpansion = (source: Source) => {
  if (isPlanBSelected.value) return

  const sourceKey = getSourceKey(source)
  expandedPlanASourceKey.value = expandedPlanASourceKey.value === sourceKey ? null : sourceKey
}

const openPlanASourceDetail = (source: Source) => {
  if (isPlanBSelected.value || isPlanCSelected.value) return

  selectedPlanASource.value = source
  expandedPlanASourceKey.value = null
  isPlanAAbstractExpanded.value = false
  isSortMenuOpen.value = false
  isFilterMenuOpen.value = false
}

const closePlanASourceDetail = () => {
  selectedPlanASource.value = null
  isPlanAAbstractExpanded.value = false
}

const clearPlanCSearchTimers = () => {
  planCSearchTimers.forEach((timer) => window.clearTimeout(timer))
  planCSearchTimers = []
}

const clearPlanCSourceLoadingTimer = () => {
  if (planCSourceLoadingTimer !== null) {
    window.clearTimeout(planCSourceLoadingTimer)
    planCSourceLoadingTimer = null
  }
}

const runPlanCSearch = () => {
  if (!hasPlanCQuery.value) return

  isPlanC5InlineLoading.value = isPlanC5Selected.value && hasPlanCResults.value

  clearPlanCSearchTimers()
  clearPlanCSourceLoadingTimer()

  isPlanCGenerating.value = true
  hasPlanCResults.value = false
  isPlanCSourceLoading.value = false
  planCGeneratingStep.value = 0
  selectedPlanCSourceIndex.value = 0
  isPlanCAbstractExpanded.value = false

  const stepDuration = 900

  planCGenerationSteps.forEach((_, index) => {
    const timer = window.setTimeout(() => {
      planCGeneratingStep.value = index + 1
    }, stepDuration * (index + 1))

    planCSearchTimers.push(timer)
  })

  const finishTimer = window.setTimeout(() => {
    isPlanCGenerating.value = false
    hasPlanCResults.value = true
    planCGeneratingStep.value = 0
    isPlanC5InlineLoading.value = false
    planCSearchTimers = []
  }, stepDuration * planCGenerationSteps.length + 500)

  planCSearchTimers.push(finishTimer)
}

const selectPlanCSource = (index: number) => {
  if (index === selectedPlanCSourceIndex.value) return

  clearPlanCSourceLoadingTimer()
  selectedPlanCSourceIndex.value = index
  if (isPlanC4LikeSelected.value) {
    requestAnimationFrame(() => {
      planCResultCardRefs.value[index]?.scrollIntoView({ behavior: 'smooth', block: 'start' })
    })
  }
  isPlanCSourceLoading.value = true
  isPlanC5InlineLoading.value = isPlanC5Selected.value
  isPlanCAbstractExpanded.value = false

  planCSourceLoadingTimer = window.setTimeout(() => {
    isPlanCSourceLoading.value = false
    isPlanC5InlineLoading.value = false
    planCSourceLoadingTimer = null
  }, 650)
}

const clearPlanCSearch = () => {
  clearPlanCSearchTimers()
  clearPlanCSourceLoadingTimer()

  planCQuery.value = ''
  isPlanCGenerating.value = false
  hasPlanCResults.value = false
  isPlanCSourceLoading.value = false
  isPlanC5InlineLoading.value = false
  planCGeneratingStep.value = 0
  selectedPlanCSourceIndex.value = 0
  isPlanCAbstractExpanded.value = false
}

const clearPlanCQueryOnly = () => {
  planCQuery.value = ''
}

const openCitationDialog = (source: Source) => {
  activeCitationSource.value = source
  selectedCitationFormat.value = 'APA'
  isProjectMenuOpen.value = false
  isSortMenuOpen.value = false
  isFilterMenuOpen.value = false
}

const openReferenceCitationDialog = () => {
  const reference = referenceCards[activeReferenceIndex.value]

  activeCitationSource.value = {
    title: reference.title,
    quotes: '1 Relevant Quote',
    quoteDetails: [
      {
        label: 'QUOTE',
        lead: reference.quoteLead,
        body: reference.quoteBody,
      },
    ],
    takeaway: `${reference.quoteLead} ${reference.quoteBody}`,
    year: reference.year,
    author: reference.author,
    type: reference.type,
    citationCount: reference.citationCount,
  }
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

const copySuggestedSentence = async (source: Source) => {
  if (!source.suggestedSentence) return

  await navigator.clipboard?.writeText(source.suggestedSentence)
}

const copyPlainText = async (text?: string) => {
  if (!text) return

  await navigator.clipboard?.writeText(text)
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

  if (referencePopoverCloseTimer !== null) {
    window.clearTimeout(referencePopoverCloseTimer)
  }

  clearPlanCSearchTimers()
  clearPlanCSourceLoadingTimer()
})

const planASources: Source[] = [
  {
    title: 'Sleep deprivation and cytokine concentrations',
    quotes: '3 Relevant Quote',
    quoteDetails: [
      {
        label: 'QUOTE 1',
        lead: 'Defines the original Fogg Behavior Model and',
        body: 'explains how motivation, ability, and prompt converge to produce behavior.',
      },
    ],
    takeaway: 'Defining the Fogg Behavior Model',
    year: '2001',
    author: 'Shearer et al.',
    type: 'Research article',
    citationCount: '1,240',
    relevanceLabel: 'Relevance Reason',
    relevanceText:
      'Defines the original Fogg Behavior Model and explains how motivation, ability, and prompt converge to produce behavior.',
    bestUsedFor: 'Defining the Fogg Behavior Model',
    suggestedSentence:
      'The Fogg Behavior Model argues that a target behavior occurs when motivation, ability, and a prompt converge at the same moment.',
    otherWays: [
      {
        title: 'Build your framework',
        sentence: 'Use Fogg’s model as the main lens for analysing digital behavior change.',
      },
      {
        title: 'Compare with other models',
        sentence: 'Contrast Fogg with COM-B or the Transtheoretical Model.',
      },
    ],
  },
  {
    title: 'Behavior design and persuasive technology in learning systems',
    quotes: '2 Relevant Quote',
    quoteDetails: [
      {
        label: 'QUOTE 1',
        lead: 'Persuasive design principles',
        body: 'help explain how prompts and motivation shape repeated learning behavior.',
      },
    ],
    takeaway: 'Connecting behavioral prompts to learning product design',
    year: '2009',
    author: 'Fogg et al.',
    type: 'Research article',
    citationCount: '986',
    relevanceLabel: 'Relevance Reason',
    relevanceText:
      'Connects behavior design principles to prompts, motivation, and ability in digital learning systems.',
    bestUsedFor: 'Explaining persuasive learning product mechanics',
    suggestedSentence:
      'Persuasive learning tools can support behavior change when prompts are timed around learner motivation and ability.',
    otherWays: [
      {
        title: 'Connect to product strategy',
        sentence: 'Use persuasive design principles to explain how learning products guide user behavior.',
      },
      {
        title: 'Support intervention design',
        sentence: 'Frame prompts as interventions that become effective when learner motivation is already present.',
      },
    ],
  },
  {
    title: 'A systematic review of habit formation in educational technology',
    quotes: '4 Relevant Quote',
    quoteDetails: [
      {
        label: 'QUOTE 1',
        lead: 'Habit formation research',
        body: 'shows that repeated cues and low-friction actions can sustain engagement over time.',
      },
    ],
    takeaway: 'Supporting claims about sustained engagement and habit formation',
    year: '2018',
    author: 'Gardner et al.',
    type: 'Systematic review',
    citationCount: '742',
    relevanceLabel: 'Relevance Reason',
    relevanceText:
      'Summarizes evidence on habit formation and the role of cues, friction, and repeated action.',
    bestUsedFor: 'Discussing long-term engagement patterns',
    suggestedSentence:
      'Habit formation studies suggest that repeated cues and low-friction actions are central to sustaining engagement.',
    otherWays: [
      {
        title: 'Explain retention patterns',
        sentence: 'Use habit formation evidence to discuss why learners return to repeated study routines.',
      },
      {
        title: 'Evaluate engagement features',
        sentence: 'Assess whether product cues reduce friction enough to support repeated learner action.',
      },
    ],
  },
  {
    title: 'Motivation, ability, and user engagement in adaptive platforms',
    quotes: '1 Relevant Quote',
    quoteDetails: [
      {
        label: 'QUOTE 1',
        lead: 'Adaptive systems',
        body: 'can improve engagement when they reduce task difficulty and provide timely prompts.',
      },
    ],
    takeaway: 'Applying motivation and ability concepts to adaptive systems',
    year: '2020',
    author: 'Kim et al.',
    type: 'Journal paper',
    citationCount: '518',
    relevanceLabel: 'Relevance Reason',
    relevanceText:
      'Applies motivation and ability concepts to adaptive educational technology and student engagement.',
    bestUsedFor: 'Framing adaptive prompt design',
    suggestedSentence:
      'Adaptive platforms can increase engagement by reducing task difficulty and presenting prompts at moments of readiness.',
    otherWays: [
      {
        title: 'Describe adaptive support',
        sentence: 'Use this source to explain how adaptive systems lower difficulty for learners.',
      },
      {
        title: 'Discuss readiness',
        sentence: 'Connect motivation and ability to the timing of automated learning prompts.',
      },
    ],
  },
  {
    title: 'Prompt timing and learner action in digital study tools',
    quotes: '2 Relevant Quote',
    quoteDetails: [
      {
        label: 'QUOTE 1',
        lead: 'Prompt timing',
        body: 'is most effective when aligned with user intent and the immediate context of action.',
      },
    ],
    takeaway: 'Explaining why prompt timing matters in study workflows',
    year: '2022',
    author: 'Wise et al.',
    type: 'Research article',
    citationCount: '365',
    relevanceLabel: 'Relevance Reason',
    relevanceText:
      'Explains how well-timed prompts can increase the likelihood of learner action in digital tools.',
    bestUsedFor: 'Describing prompt timing in study experiences',
    suggestedSentence:
      'Well-timed prompts can increase learner action when they appear in the immediate context of a study task.',
    otherWays: [
      {
        title: 'Analyze study workflows',
        sentence: 'Use prompt timing to explain where nudges should appear inside a study flow.',
      },
      {
        title: 'Justify notification design',
        sentence: 'Support claims about why context-aware notifications can improve task completion.',
      },
    ],
  },
]

const planCSources: Source[] = planASources.map((source) => ({
  ...source,
  quoteDetails: source.quoteDetails.map((quote) => ({ ...quote })),
  otherWays: source.otherWays?.map((way) => ({ ...way })),
}))

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
            <span>{{ selectedProjectNav }}</span>
            <svg viewBox="0 0 20 20" aria-hidden="true">
              <path d="m6.5 8.2 3.5 3.5 3.5-3.5" />
            </svg>
          </button>
          <div v-if="isProjectMenuOpen" class="project-menu" role="menu">
            <button
              v-for="project in projectNavItems"
              :key="project"
              type="button"
              role="menuitemradio"
              :aria-checked="selectedProjectNav === project"
              :class="{ active: selectedProjectNav === project }"
              @click.stop="selectProjectNav(project)"
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

    <main v-if="isSidebarExpandProjectSelected" class="sidebar-expand-page" aria-label="AI Detector with expanded sidebar">
      <div class="detector-aurora" aria-hidden="true"></div>

      <header class="detector-header">
        <a class="detector-logo" href="#" aria-label="Solvely.ai">
          <span class="detector-logo-mark" aria-hidden="true">
            <svg viewBox="0 0 24 24">
              <path d="M7 4.5h10A2.5 2.5 0 0 1 19.5 7v10a2.5 2.5 0 0 1-2.5 2.5H7A2.5 2.5 0 0 1 4.5 17V7A2.5 2.5 0 0 1 7 4.5Z" />
              <path d="M8.2 12c1.2-2.2 2.5-3.3 3.8-3.3 1 0 1.8.5 2.6 1.5M15.8 12c-1.2 2.2-2.5 3.3-3.8 3.3-1 0-1.8-.5-2.6-1.5" />
              <path d="m7.5 8.9 2.1 2M14.4 13.1l2.1 2M16.5 8.9l-2.1 2M9.6 13.1l-2.1 2" />
            </svg>
          </span>
          <span>Solvely.ai</span>
        </a>

        <nav class="detector-site-nav" aria-label="Solvely navigation">
          <a href="#">
            AI Study Tools
            <svg viewBox="0 0 20 20" aria-hidden="true"><path d="m6.5 8.2 3.5 3.5 3.5-3.5" /></svg>
          </a>
          <a href="#">
            Resources
            <svg viewBox="0 0 20 20" aria-hidden="true"><path d="m6.5 8.2 3.5 3.5 3.5-3.5" /></svg>
          </a>
          <a href="#">Pricing</a>
        </nav>

        <div class="detector-auth">
          <a class="detector-try" href="#">Try For Free</a>
          <a href="#">Log in</a>
        </div>
      </header>

      <section class="detector-hero">
        <div class="detector-trust-badge">
          <span class="detector-avatar-stack" aria-hidden="true">
            <span></span>
            <span></span>
            <span></span>
          </span>
          <span>Trusted by <strong>24M+</strong> users · <strong>39K+</strong> are checking their text right now</span>
        </div>
        <h1><span>AI Detector</span> with Fewer False Positives</h1>
        <p>Run multi-engine AI detection with 98.7% accuracy, then humanize in one click</p>
      </section>

      <section class="detector-tool-card" :class="{ 'rail-expanded': isDetectorRailExpanded }" aria-label="AI detector input and report">
        <aside
          class="detector-tool-rail"
          aria-label="Tool switcher"
          @mouseenter="isDetectorRailExpanded = true"
          @mouseleave="isDetectorRailExpanded = false"
          @focusin="isDetectorRailExpanded = true"
          @focusout="isDetectorRailExpanded = false"
        >
          <button class="active" type="button" aria-label="AI Detector">
            <span class="detector-tool-icon" aria-hidden="true"><svg viewBox="0 0 20 20"><path d="M10 4.3a5.7 5.7 0 1 0 5.7 5.7" /><path d="M10 7.4a2.6 2.6 0 1 0 2.6 2.6" /><path d="M10 10h6.2M14.2 7.6l2 2.4-2 2.4" /></svg></span>
            <span class="detector-tool-label">AI Detector</span>
          </button>
          <button type="button" aria-label="AI Humanizer">
            <span class="detector-tool-icon" aria-hidden="true"><svg viewBox="0 0 20 20"><path d="M7.2 14.8v-1.2a4.4 4.4 0 0 1-1.7-3.5V8a4.5 4.5 0 1 1 9 0v6.8" /><path d="M8.4 15.5h5.2M8.9 18h4.2" /></svg></span>
            <span class="detector-tool-label">AI Humanizer</span>
          </button>
          <button type="button" aria-label="Plagiarism Checker">
            <span class="detector-tool-icon" aria-hidden="true"><svg viewBox="0 0 20 20"><path d="M6.4 3.8h5.9l2.9 2.9v9.5H6.4z" /><path d="M12.1 3.9v3h3M8.4 10h4M8.4 12.7h4" /></svg></span>
            <span class="detector-tool-label">Plagiarism Checker</span>
          </button>
          <button type="button" aria-label="Citation Generator">
            <span class="detector-tool-icon" aria-hidden="true"><svg viewBox="0 0 20 20"><path d="M7 7.5H5.8A1.8 1.8 0 0 0 4 9.3v1.2A1.8 1.8 0 0 0 5.8 12H7v-1.6H5.9V9.1H7zM13 7.5h1.2A1.8 1.8 0 0 1 16 9.3v1.2a1.8 1.8 0 0 1-1.8 1.5H13v-1.6h1.1V9.1H13z" /></svg></span>
            <span class="detector-tool-label">Citation Generator</span>
          </button>
          <button type="button" aria-label="AI Paraphraser">
            <span class="detector-tool-icon" aria-hidden="true"><svg viewBox="0 0 20 20"><path d="M5 7h8.8l-2-2M15 13H6.2l2 2" /></svg></span>
            <span class="detector-tool-label">AI Paraphraser</span>
          </button>
          <button type="button" aria-label="AI Research">
            <span class="detector-tool-icon" aria-hidden="true"><svg viewBox="0 0 20 20"><path d="M8.8 3.8h2.4M10 3.8v4.4l-3.7 6.5a1.2 1.2 0 0 0 1 1.8h5.4a1.2 1.2 0 0 0 1-1.8L10 8.2" /><path d="M7.7 12.2h4.6" /></svg></span>
            <span class="detector-tool-label">AI Research</span>
          </button>
          <button type="button" aria-label="Word Counter">
            <span class="detector-tool-icon detector-word-counter-icon" aria-hidden="true"><svg viewBox="0 0 20 20"><path d="M6.4 5.2 5 14.8M11.5 5.2l-1.4 9.6M3.8 8.6h10.8M3.1 11.4h10.8" /></svg></span>
            <span class="detector-tool-label">Word Counter</span>
          </button>
        </aside>

        <div class="detector-input-pane">
          <div class="detector-pane-header">
            <h2>Original text:</h2>
            <button type="button" class="detector-history">
              <span aria-hidden="true">↺</span>
              History
            </button>
          </div>
          <div class="detector-filled-input">
            <p>
              <span class="detector-flagged-copy">Spring is a beautiful season that brings warmth and new life to the environment.</span>
              <span class="detector-more-flags">+3 more</span>
              During this time, flowers begin to bloom and trees turn green again. The weather becomes more pleasant, and people often spend more time outdoors. Many individuals feel happier and more energetic as the days grow longer.
            </p>
          </div>
          <footer class="detector-input-footer">
            <span>130 words</span>
            <div class="detector-input-actions">
              <button type="button" class="detector-clear-button">Clear</button>
              <button type="button" class="detector-check-button">
                <svg viewBox="0 0 20 20" aria-hidden="true"><path d="m10 3.8 1.1 3.8 3.7 1.4-3.7 1.4L10 14.2l-1.1-3.8L5.2 9l3.7-1.4z" /></svg>
                Check for AI
              </button>
            </div>
          </footer>
        </div>

        <div class="detector-card-divider" aria-hidden="true"></div>

        <div class="detector-report-pane">
          <div class="detector-pane-header">
            <h2>AI detection report</h2>
            <div class="detector-style-selector" aria-hidden="true">
              <span>Style:</span>
              <button type="button">
                Original
                <svg viewBox="0 0 20 20"><path d="m6.5 8.2 3.5 3.5 3.5-3.5" /></svg>
              </button>
            </div>
          </div>
          <div class="detector-result-content">
            <div class="detector-result-warning">
              <svg viewBox="0 0 20 20" aria-hidden="true">
                <circle cx="10" cy="10" r="7" />
                <path d="M10 6.3v4.6M10 13.8v.1" />
              </svg>
              <span>You text has been changed. Run again to see the updated result</span>
            </div>

            <section class="detector-score-card" aria-label="AI likelihood score">
              <div class="detector-score-gauge" aria-hidden="true">
                <svg viewBox="0 0 154 86">
                  <path d="M12 73a65 65 0 0 1 130 0" />
                </svg>
                <div>
                  <strong>100<span>%</span></strong>
                  <small>AI possibility</small>
                </div>
              </div>
              <div class="detector-score-summary">
                <div class="detector-score-heading">
                  <strong>Your text appears</strong>
                  <span>High AI Likelihood</span>
                </div>
                <button type="button" class="detector-remove-ai">
                  <svg viewBox="0 0 20 20" aria-hidden="true">
                    <path d="m8.3 4.3 5 5M6.8 8l5.2 5.2M5.2 9.5l5.3 5.3M4.6 10.1l-.8 4.5 4.5-.8" />
                    <path d="m9.1 3.5 7.4 7.4-2.3 2.3-7.4-7.4z" />
                  </svg>
                  Remove AI
                </button>
              </div>
            </section>

            <section class="detector-cross-check" aria-label="Cross check results">
              <h3>Cross check results</h3>
              <div class="detector-cross-list">
                <div class="detector-cross-row">
                  <div class="detector-engine-name">
                    <span class="detector-engine-icon fingerprint" aria-hidden="true">
                      <svg viewBox="0 0 24 24"><path d="M12 3.7a7 7 0 0 0-7 7M12 6.6a4.1 4.1 0 0 0-4.1 4.1c0 3.1-.7 5.1-1.8 6.7M12 9.3a1.4 1.4 0 0 0-1.4 1.4c0 4.2-1.1 7-2.5 9M14.8 4.3a7 7 0 0 1 4.2 6.4c0 3.5-.6 6.4-1.8 9M14.9 7.8a4.1 4.1 0 0 1 1.2 2.9c0 4-.6 6.8-1.7 9.4M13.3 12.1c-.1 3.6-.6 6.1-1.4 8" /></svg>
                    </span>
                    <strong>ZeroGPT</strong>
                  </div>
                  <span class="detector-cross-score">80% AI possibility</span>
                  <span class="detector-likelihood-pill">High likelihood</span>
                </div>

                <div class="detector-cross-row locked">
                  <div class="detector-engine-name">
                    <span class="detector-engine-icon turnitin" aria-hidden="true">
                      <svg viewBox="0 0 24 24"><path d="M7 4.5h8.8v15H7z" /><path d="m10.1 8.1-2.4 2.4 2.4 2.4M7.7 10.5h9.2M13.9 8.1l2.4 2.4-2.4 2.4" /></svg>
                    </span>
                    <strong>Turnitin</strong>
                    <span class="detector-pro-tag">Pro</span>
                  </div>
                  <span class="detector-cross-score blurred">20% AI possibility</span>
                  <button type="button" class="detector-unlock-button">
                    <svg viewBox="0 0 16 16" aria-hidden="true"><rect x="3.4" y="7" width="9.2" height="6.2" rx="1.4" /><path d="M5.3 7V5.2a2.7 2.7 0 0 1 5.4 0V7" /></svg>
                    Unlock
                  </button>
                </div>

                <div class="detector-cross-row locked">
                  <div class="detector-engine-name">
                    <span class="detector-engine-icon gptzero" aria-hidden="true">
                      <svg viewBox="0 0 24 24"><circle cx="12" cy="12" r="7.4" /><circle cx="12" cy="12" r="3.8" /><path d="M12 12h7.5M17 9.3l2.5 2.7-2.5 2.7" /></svg>
                    </span>
                    <strong>GPTZero</strong>
                    <span class="detector-pro-tag">Pro</span>
                  </div>
                  <span class="detector-cross-score blurred">20% AI possibility</span>
                  <button type="button" class="detector-unlock-button">
                    <svg viewBox="0 0 16 16" aria-hidden="true"><rect x="3.4" y="7" width="9.2" height="6.2" rx="1.4" /><path d="M5.3 7V5.2a2.7 2.7 0 0 1 5.4 0V7" /></svg>
                    Unlock
                  </button>
                </div>
              </div>
            </section>
          </div>
        </div>
      </section>

      <footer class="detector-proof-row">
        <div class="detector-rating">
          <span aria-hidden="true">★★★★★</span>
          <p><strong>4.8/5</strong> Based on on <strong>39K</strong> ratings on the App Store</p>
        </div>
        <div class="detector-engines">
          <span>Double check by:</span>
          <strong>ZeroGPT</strong>
          <strong>GPTZero</strong>
          <strong>turnitin</strong>
        </div>
      </footer>
    </main>

    <main v-else class="app-shell">
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

      <section class="workspace-card" :class="{ 'workspace-card-plan-c': isPlanCSelected }" aria-label="AI Research">
        <template v-if="isPlanCSelected">
          <div class="plan-c-tabs" aria-label="AI Writing tool tabs">
            <button v-for="tab in tabs" :key="tab" :class="{ active: tab === 'AI Research' }" type="button">
              {{ tab }}
            </button>
          </div>

          <div
            class="plan-c-input-page"
            :class="{
              'plan-c-input-page-no-divider': isPlanC4LikeSelected,
              'plan-c4-page': isPlanC4Selected,
              'plan-c5-page': isPlanC5Selected,
              'plan-c5-page-active': isPlanC5Selected && isPlanCSearchActive,
            }"
          >
            <section
              class="plan-c-input-pane"
              :class="{ 'plan-c5-input-pane': isPlanC5Selected && !isPlanCSearchActive }"
            >
              <div class="plan-c-input-header">
                <button
                  v-if="isPlanC5Selected && isPlanCSearchActive"
                  class="plan-c5-back-button"
                  type="button"
                  @click="clearPlanCSearch"
                >
                  <svg viewBox="0 0 14 14" aria-hidden="true">
                    <path d="M8.8 3.2 5 7l3.8 3.8" />
                  </svg>
                  Back
                </button>
                <h2 v-else>{{ isPlanCGenerating ? 'Finding relevant sources...' : isPlanCSearchActive || isPlanC5Selected ? 'Your Search' : 'Find Sources for Your Paper' }}</h2>
                <div class="plan-c-header-actions">
                  <button class="plan-c-saved" type="button">
                    <img :src="collectIcon" alt="" />
                    <span>Saved</span>
                    <strong>0</strong>
                  </button>
                  <button class="plan-c-history" type="button">
                    <img :src="historyIcon" alt="" />
                    <span>History</span>
                  </button>
                </div>
              </div>

              <template v-if="!isPlanCSearchActive">
                <template v-if="isPlanC5Selected">
                  <div class="plan-c5-home-body">
                    <div class="plan-c5-search-panel">
                      <textarea
                          v-model="planCQuery"
                          class="plan-c5-search-textarea"
                          aria-label="Research topic"
                          placeholder="Enter your topic, research question, or claim"
                          @keydown.enter.prevent="runPlanCSearch"
                      ></textarea>
                      <div class="plan-c5-search-footer">
                        <span aria-hidden="true">130 words</span>
                        <button
                          class="plan-c-search-button"
                          :class="{ active: hasPlanCQuery }"
                          type="button"
                          :disabled="!hasPlanCQuery"
                          @click="runPlanCSearch"
                        >
                          <svg viewBox="0 0 20 20" aria-hidden="true">
                            <path d="M9 15.5a6.5 6.5 0 1 1 0-13 6.5 6.5 0 0 1 0 13Z" />
                            <path d="m13.8 13.8 3.4 3.4" />
                          </svg>
                          Search
                        </button>
                      </div>
                    </div>

                    <div class="plan-c5-examples">
                      <p>Or, try searching for:</p>
                      <div class="plan-c3-example-grid">
                        <button
                          v-for="card in planC3PromptCards"
                          :key="card.kind"
                          class="plan-c3-example-card"
                          type="button"
                          @click="planCQuery = card.text"
                        >
                          <span class="plan-c3-example-icon" :data-icon="card.icon" aria-hidden="true">
                            <svg v-if="card.icon === 'hash'" viewBox="0 0 20 20">
                              <path d="M7.2 3.8 5.8 16.2M14.2 3.8l-1.4 12.4M4 8h12M3.5 12h12" />
                            </svg>
                            <svg v-else-if="card.icon === 'notebook'" viewBox="0 0 20 20">
                              <path d="M6 4.2h9.2v11.6H6zM4.8 5.5h2.4M4.8 8.2h2.4M4.8 10.9h2.4M4.8 13.6h2.4" />
                              <path d="M9.2 7.2H13M9.2 10H13" />
                            </svg>
                            <svg v-else viewBox="0 0 20 20">
                              <path d="M6.2 6.9a3.9 3.9 0 0 1 7.6 1.2c0 2.8-3 2.9-3.6 4.9M10.1 16.2h.01" />
                              <path d="M4.5 3.8h11v12.4h-11z" />
                            </svg>
                          </span>
                          <strong>{{ card.kind }}</strong>
                          <span>{{ card.text }}</span>
                        </button>
                      </div>
                    </div>
                  </div>
                </template>

                <template v-else-if="isPlanCAdvancedSelected">
                  <div class="plan-c3-prompt-body">
                    <textarea
                      v-model="planCQuery"
                      class="plan-c3-prompt-input"
                      aria-label="Research topic"
                      placeholder="Enter your topic, research question or assignment prompt to find credible sources..."
                    ></textarea>

                    <div class="plan-c3-examples">
                      <p>Or, try searching for:</p>
                      <div class="plan-c3-example-grid">
                        <button
                          v-for="card in planC3PromptCards"
                          :key="card.kind"
                          class="plan-c3-example-card"
                          type="button"
                          @click="planCQuery = card.text"
                        >
                          <span class="plan-c3-example-icon" :data-icon="card.icon" aria-hidden="true">
                            <svg v-if="card.icon === 'hash'" viewBox="0 0 20 20">
                              <path d="M7.2 3.8 5.8 16.2M14.2 3.8l-1.4 12.4M4 8h12M3.5 12h12" />
                            </svg>
                            <svg v-else-if="card.icon === 'notebook'" viewBox="0 0 20 20">
                              <path d="M6 4.2h9.2v11.6H6zM4.8 5.5h2.4M4.8 8.2h2.4M4.8 10.9h2.4M4.8 13.6h2.4" />
                              <path d="M9.2 7.2H13M9.2 10H13" />
                            </svg>
                            <svg v-else viewBox="0 0 20 20">
                              <path d="M6.2 6.9a3.9 3.9 0 0 1 7.6 1.2c0 2.8-3 2.9-3.6 4.9M10.1 16.2h.01" />
                              <path d="M4.5 3.8h11v12.4h-11z" />
                            </svg>
                          </span>
                          <strong>{{ card.kind }}</strong>
                          <span>{{ card.text }}</span>
                        </button>
                      </div>
                    </div>
                  </div>

                  <div class="plan-c-input-footer plan-c3-input-footer">
                    <span>130 words</span>
                    <button
                      class="plan-c-search-button"
                      :class="{ active: hasPlanCQuery }"
                      type="button"
                      :disabled="!hasPlanCQuery"
                      @click="runPlanCSearch"
                    >
                      <svg viewBox="0 0 20 20" aria-hidden="true">
                        <path d="M9 15.5a6.5 6.5 0 1 1 0-13 6.5 6.5 0 0 1 0 13Z" />
                        <path d="m13.8 13.8 3.4 3.4" />
                      </svg>
                      Search
                    </button>
                  </div>
                </template>

                <template v-else>
                  <textarea
                    v-model="planCQuery"
                    class="plan-c-topic-input"
                    aria-label="Research topic"
                    placeholder="Enter your topic, research question, or claim"
                  ></textarea>

                  <div class="plan-c-suggestions">
                    <button
                      v-for="suggestion in planCSuggestions"
                      :key="suggestion"
                      type="button"
                      @click="planCQuery = suggestion"
                    >
                      <span>{{ suggestion }}</span>
                      <svg viewBox="0 0 16 16" aria-hidden="true">
                        <path d="M5 11 11 5M7 5h4v4" />
                      </svg>
                    </button>
                  </div>

                  <div class="plan-c-input-footer">
                    <span>130 words</span>
                    <button
                      class="plan-c-search-button"
                      :class="{ active: hasPlanCQuery }"
                      type="button"
                      :disabled="!hasPlanCQuery"
                      @click="runPlanCSearch"
                    >
                      <svg viewBox="0 0 20 20" aria-hidden="true">
                        <path d="M9 15.5a6.5 6.5 0 1 1 0-13 6.5 6.5 0 0 1 0 13Z" />
                        <path d="m13.8 13.8 3.4 3.4" />
                      </svg>
                      Search
                    </button>
                  </div>
                </template>
              </template>

              <template v-else>
                <div class="plan-c-result-search" :class="{ 'plan-c-result-search-loading': isPlanCGenerating }">
                  <div class="plan-c-query-pill">
                    <input
                      v-model="planCQuery"
                      aria-label="Edit research search"
                      :disabled="isPlanCGenerating"
                      @keydown.enter.prevent="runPlanCSearch"
                    />
                    <button
                      class="plan-c-clear-search"
                      type="button"
                      aria-label="Clear search"
                      @click="clearPlanCQueryOnly"
                    >
                      <svg viewBox="0 0 20 20" aria-hidden="true">
                        <path d="M5.5 5.5 14.5 14.5M14.5 5.5 5.5 14.5" />
                      </svg>
                    </button>
                  </div>
                  <button
                    class="plan-c-search-button plan-c-search-icon-button active"
                    :class="{ loading: isPlanCGenerating }"
                    type="button"
                    aria-label="Search"
                    :disabled="isPlanCGenerating || !hasPlanCQuery"
                    @click="runPlanCSearch"
                  >
                    <svg v-if="isPlanCGenerating" class="plan-c-loading-search-icon" viewBox="0 0 20 20" aria-hidden="true">
                      <path d="M10 3.2a6.8 6.8 0 1 1-6.8 6.8" />
                    </svg>
                    <svg v-else viewBox="0 0 20 20" aria-hidden="true">
                      <path d="M9 15.5a6.5 6.5 0 1 1 0-13 6.5 6.5 0 0 1 0 13Z" />
                      <path d="m13.8 13.8 3.4 3.4" />
                    </svg>
                  </button>
                </div>

                <div v-if="isPlanCGenerating" class="plan-c-generating-steps" aria-live="polite">
                  <PlanC4Loading
                    v-if="isPlanC4LikeSelected"
                    :steps="planCGenerationSteps"
                    :current-step="planCGeneratingStep"
                  />
                  <PlanC3Loading
                    v-else-if="isPlanC3Selected"
                    :steps="planCGenerationSteps"
                    :current-step="planCGeneratingStep"
                  />
                  <PlanC2Loading
                    v-else-if="isPlanC2Selected"
                    :steps="planCGenerationSteps"
                    :current-step="planCGeneratingStep"
                  />
                  <template v-else>
                    <div
                      v-for="(step, index) in planCGenerationSteps"
                      :key="step"
                      class="plan-c-generating-step"
                      :class="{
                        complete: index < planCGeneratingStep,
                        current: index === planCGeneratingStep,
                        pending: index > planCGeneratingStep,
                      }"
                    >
                      <span class="plan-c-step-icon">
                        <svg v-if="index < planCGeneratingStep" viewBox="0 0 20 20" aria-hidden="true">
                          <path d="m5.5 10.2 3 3 6-6.4" />
                        </svg>
                        <svg v-else viewBox="0 0 20 20" aria-hidden="true">
                          <path d="M10 3a7 7 0 1 1-7 7" />
                        </svg>
                      </span>
                      <span>{{ step }}</span>
                    </div>
                  </template>
                </div>

                <template v-else>
                  <div class="plan-c-results-bar" :class="{ 'plan-c5-results-bar': isPlanC5Selected }">
                    <strong>About 128 results</strong>
                    <div>
                      <div ref="filterMenuRef" class="filter-control plan-c-filter-control">
                        <button
                          type="button"
                          :class="{ active: isFilterActive }"
                          :aria-expanded="isFilterMenuOpen"
                          aria-haspopup="dialog"
                          @click.stop="toggleFilterMenu"
                          @keydown.esc="isFilterMenuOpen = false"
                        >
                          <svg viewBox="0 0 20 20" aria-hidden="true">
                            <path d="M4 5.5h12M6.5 10h7M8.5 14.5h3" />
                          </svg>
                          <span v-if="!isPlanC5Selected">Filters</span>
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
                                :class="{ active: pendingResourceType === type }"
                                @click="pendingResourceType = type"
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
                            <button class="confirm-filter" type="button" @click="confirmFilters">
                              Confirm
                            </button>
                          </div>
                        </div>
                      </div>
                      <div ref="sortMenuRef" class="sort-control plan-c-sort-control">
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
                          <svg v-if="!isPlanC5Selected" class="chevron" viewBox="0 0 20 20" aria-hidden="true">
                            <path d="m6.5 8.2 3.5 3.5 3.5-3.5" />
                          </svg>
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
                    </div>
                  </div>

                  <div class="plan-c-result-list">
                    <article
                      v-for="(source, index) in planCResultSources"
                      :key="`${source.title}-${index}`"
                      :ref="(element) => setPlanCResultCardRef(element, index)"
                      class="plan-c-result-card"
                      :class="{ selected: selectedPlanCSourceIndex === index }"
                      @click="selectPlanCSource(index)"
                    >
                      <h3>{{ getPlanCDisplayTitle(source) }}</h3>
                      <p v-if="isPlanC4LikeSelected" class="plan-c-result-meta">
                        {{ source.year }} &middot; {{ source.author }} &middot; {{ source.venue }} &middot; {{ source.type }}
                      </p>
                      <div v-if="isPlanC4Selected" class="plan-c4-card-tags">
                        <span v-for="tag in getPlanC4CardTags(source)" :key="tag">{{ tag }}</span>
                      </div>
                      <div v-else-if="isPlanC5Selected" class="plan-c4-card-tags">
                        <span v-for="tag in getPlanC5CardTags(source)" :key="tag">{{ tag }}</span>
                      </div>
                      <div v-if="!isPlanC4LikeSelected" class="plan-c-result-tags">
                        <span v-for="tag in source.tags" :key="tag">{{ tag }}</span>
                      </div>
                      <p class="plan-c-snapshot"><strong>SNAPSHOT</strong> &middot; {{ source.snapshot }}</p>
                      <p v-if="!isPlanC4LikeSelected" class="plan-c-result-meta">
                        {{ source.year }} &middot; {{ source.author }} &middot; {{ source.venue }} &middot; {{ source.type }}
                      </p>
                      <div v-if="isPlanC4LikeSelected" class="plan-c4-useful-tag">
                        <span class="plan-c4-useful-icon" aria-hidden="true">💡</span>
                        <span>{{ getPlanCUsefulFirstSentence(source) }}.</span>
                        <svg viewBox="0 0 20 20" aria-hidden="true">
                          <path d="M4 10h12" />
                          <path d="m11 5 5 5-5 5" />
                        </svg>
                      </div>
                    </article>
                  </div>
                </template>
              </template>
            </section>

            <section
              v-if="!(isPlanC5Selected && !isPlanCSearchActive)"
              class="plan-c-empty-pane"
              :class="{
                'plan-c-details-pane': hasPlanCResults,
                'plan-c-generating-pane': isPlanCGenerating,
                'plan-c-empty-pane-c3': isPlanC3Selected && !isPlanCSearchActive,
                'plan-c-empty-pane-c4': isPlanC4LikeSelected,
              }"
            >
              <div v-if="!isPlanCSearchActive && isPlanC4LikeSelected" class="plan-c-result-empty-c4">
                <header>
                  <h2>Source Details</h2>
                </header>
                <div class="plan-c-result-empty-c4-body">
                  <div class="plan-c-result-empty-c4-icon" aria-hidden="true">🔍</div>
                  <p>Waite for Searching...</p>
                </div>
              </div>
              <div
                v-else-if="isPlanC4LikeSelected && (isPlanCGenerating || isPlanCSourceLoading)"
                class="plan-c4-source-panel"
                :class="{ 'plan-c5-loading-panel': isPlanC5Selected && !isPlanC5InlineLoading }"
                aria-hidden="true"
              >
                <header>
                  <h2>Source Details</h2>
                </header>
                <div
                  class="plan-c4-source-body plan-c4-source-skeleton"
                  :class="{ 'plan-c5-loading-body': isPlanC5Selected && !isPlanC5InlineLoading }"
                >
                  <div class="plan-c4-skeleton-title"></div>
                  <div class="plan-c4-skeleton-meta"></div>
                  <div class="plan-c4-skeleton-actions">
                    <span></span>
                    <span></span>
                    <span></span>
                  </div>
                  <div class="plan-c4-divider"></div>
                  <div class="plan-c4-skeleton-section"></div>
                  <div class="plan-c4-skeleton-line wide"></div>
                  <div class="plan-c4-skeleton-line"></div>
                  <div class="plan-c4-skeleton-line medium"></div>
                  <div class="plan-c4-skeleton-section findings"></div>
                  <div class="plan-c4-skeleton-card"></div>
                  <div class="plan-c4-skeleton-card short"></div>
                  <div class="plan-c4-skeleton-section abstract"></div>
                  <div class="plan-c4-skeleton-line wide"></div>
                  <div class="plan-c4-skeleton-line medium"></div>
                </div>
              </div>
              <div
                v-else-if="!isPlanCSearchActive"
                class="plan-c-empty-state"
                :class="{ 'plan-c-empty-state-c3': isPlanC3Selected }"
              >
                <div class="plan-c-empty-icon" aria-hidden="true">
                  <svg v-if="isPlanC3Selected" viewBox="0 0 32 32">
                    <path d="M14.2 23.4a9.2 9.2 0 1 1 0-18.4 9.2 9.2 0 0 1 0 18.4Z" />
                    <path d="m20.8 20.8 6.2 6.2" />
                  </svg>
                  <template v-else>🔍</template>
                </div>
                <div>
                  <h2>Search to Unlock Source Details</h2>
                  <p>Run a search on the left to explore abstracts, key findings, and useful metrics for your paper.</p>
                </div>
              </div>
              <div v-else-if="isPlanCGenerating && isPlanCAdvancedSelected" class="plan-c-skeleton-details" aria-hidden="true">
                <div class="plan-c-skeleton-line short"></div>
                <div class="plan-c-skeleton-divider"></div>
                <div class="plan-c-skeleton-line title"></div>
                <div class="plan-c-skeleton-line title second"></div>
                <div class="plan-c-skeleton-line meta"></div>
                <div class="plan-c-skeleton-actions">
                  <span></span>
                  <span></span>
                  <span></span>
                </div>
                <div class="plan-c-skeleton-divider"></div>
                <div class="plan-c-skeleton-line section"></div>
                <div class="plan-c-skeleton-line body"></div>
                <div class="plan-c-skeleton-line body wide"></div>
                <div class="plan-c-skeleton-line body medium"></div>
                <div class="plan-c-skeleton-line section lower"></div>
                <div class="plan-c-skeleton-list">
                  <span></span>
                  <span></span>
                  <span></span>
                </div>
                <div class="plan-c-skeleton-card"></div>
              </div>
              <div v-else-if="isPlanCGenerating && !isPlanC2Selected" class="plan-c-generating-state" aria-live="polite">
                <div class="plan-c-loader" aria-hidden="true">
                  <span></span>
                  <span></span>
                  <span></span>
                </div>
                <div>
                  <h2>Generating sources...</h2>
                  <p>Analyzing your topic, ranking relevant papers, and preparing source details.</p>
                </div>
              </div>
              <div v-else-if="isPlanCGenerating" class="plan-c-generating-placeholder" aria-hidden="true"></div>
              <div v-else-if="isPlanCSourceLoading" class="plan-c-skeleton-details" aria-hidden="true">
                <div class="plan-c-skeleton-line short"></div>
                <div class="plan-c-skeleton-divider"></div>
                <div class="plan-c-skeleton-line title"></div>
                <div class="plan-c-skeleton-line title second"></div>
                <div class="plan-c-skeleton-line meta"></div>
                <div class="plan-c-skeleton-actions">
                  <span></span>
                  <span></span>
                  <span></span>
                </div>
                <div class="plan-c-skeleton-divider"></div>
                <div class="plan-c-skeleton-line section"></div>
                <div class="plan-c-skeleton-line body"></div>
                <div class="plan-c-skeleton-line body wide"></div>
                <div class="plan-c-skeleton-line body medium"></div>
                <div class="plan-c-skeleton-line section lower"></div>
                <div class="plan-c-skeleton-list">
                  <span></span>
                  <span></span>
                  <span></span>
                </div>
                <div class="plan-c-skeleton-card"></div>
              </div>
              <div v-else-if="isPlanC4LikeSelected" class="plan-c4-source-panel">
                <header>
                  <h2>Source Details</h2>
                </header>
                <div class="plan-c4-source-body">
                  <section class="plan-c4-detail-summary">
                    <h3>{{ getPlanCDisplayTitle(selectedPlanCSource) }}</h3>
                    <p>{{ selectedPlanCSource.year }} &middot; {{ selectedPlanCSource.detailAuthor }} &middot; {{ selectedPlanCSource.venue }} &middot; {{ selectedPlanCSource.type }}</p>
                    <div class="plan-c4-detail-actions">
                      <button type="button"><img :src="citeIcon" alt="" />Cite</button>
                      <button v-if="!isPlanC5Selected" type="button">
                        <svg viewBox="0 0 20 20" aria-hidden="true">
                          <path d="M8.5 11.5 11.5 8.5M7.2 8.8l-1.1 1.1a3 3 0 0 0 4.2 4.2l1.1-1.1M12.8 11.2l1.1-1.1a3 3 0 0 0-4.2-4.2L8.6 7" />
                        </svg>
                        DOI
                      </button>
                      <button type="button"><img :src="collectIcon" alt="" />Save</button>
                    </div>
                  </section>

                  <div class="plan-c4-divider"></div>

                  <section class="plan-c4-detail-section">
                    <h3>Why this source is useful</h3>
                    <p>{{ selectedPlanCSource.usefulSummary }}</p>
                  </section>

                  <section class="plan-c4-detail-section plan-c4-findings-section">
                    <h3>Key Findings (2)</h3>
                    <div class="plan-c4-finding-list">
                      <article
                        v-for="(finding, index) in selectedPlanCSource.findings"
                        :key="`${selectedPlanCSource.title}-${finding.text}`"
                        class="plan-c4-finding-card"
                      >
                        <div class="plan-c4-finding-quote">
                          <span class="plan-c4-finding-icon" aria-hidden="true"></span>
                          <span class="plan-c4-finding-rule" aria-hidden="true"></span>
                          <p>
                            <strong>Abstract</strong>
                            <span>&middot;</span>
                            “{{ finding.quote }}”
                          </p>
                        </div>
                        <p class="plan-c4-finding-text">
                          <strong>Finding {{ index + 1 }}</strong>
                          <span>&middot;</span>
                          {{ finding.text }}
                        </p>
                      </article>
                    </div>
                  </section>

                  <section class="plan-c4-detail-section plan-c4-abstract-section">
                    <h3>Abstract</h3>
                    <p
                      v-if="isPlanC5Selected"
                      class="plan-c4-abstract"
                    >
                      {{ selectedPlanCSource.abstractIntro }} {{ selectedPlanCSource.abstractMore }}
                    </p>
                    <p v-else class="plan-c4-abstract" :class="{ collapsed: !isPlanCAbstractExpanded }">
                      {{ selectedPlanCSource.abstractIntro }}
                      <template v-if="isPlanCAbstractExpanded">{{ ` ${selectedPlanCSource.abstractMore}` }}</template>
                      <button type="button" @click="isPlanCAbstractExpanded = !isPlanCAbstractExpanded">
                        {{ isPlanCAbstractExpanded ? 'Show Less' : 'Show More' }}
                      </button>
                    </p>
                  </section>
                </div>
              </div>
              <div v-else class="plan-c-source-details">
                <header>
                  <h2>Source Details</h2>
                </header>
                <section class="plan-c-detail-summary">
                  <h3>{{ getPlanCDisplayTitle(selectedPlanCSource) }}</h3>
                  <p>{{ selectedPlanCSource.year }} &middot; {{ selectedPlanCSource.detailAuthor }} &middot; {{ selectedPlanCSource.venue }} &middot; {{ selectedPlanCSource.type }}</p>
                  <div class="plan-c-detail-actions">
                    <button type="button"><img :src="citeIcon" alt="" />Cite</button>
                    <button type="button"><svg viewBox="0 0 20 20" aria-hidden="true"><path d="M8.5 11.5 11.5 8.5M7.2 8.8l-1.1 1.1a3 3 0 0 0 4.2 4.2l1.1-1.1M12.8 11.2l1.1-1.1a3 3 0 0 0-4.2-4.2L8.6 7" /></svg>DOI</button>
                    <button type="button"><img :src="collectIcon" alt="" />Save</button>
                  </div>
                </section>
                <section class="plan-c-detail-section">
                  <h3>Abstract</h3>
                  <p class="plan-c-abstract" :class="{ collapsed: !isPlanCAbstractExpanded }">
                    {{ selectedPlanCSource.abstractIntro }}
                    <template v-if="isPlanCAbstractExpanded">{{ ` ${selectedPlanCSource.abstractMore}` }}</template>
                    <button type="button" @click="isPlanCAbstractExpanded = !isPlanCAbstractExpanded">
                      {{ isPlanCAbstractExpanded ? 'Show Less' : 'Show More' }}
                    </button>
                  </p>
                </section>
                <section v-if="isPlanCAdvancedSelected" class="plan-c-detail-section plan-c-snapshot-section">
                  <h3>Snapshot</h3>
                  <div class="plan-c-snapshot-table" role="table" aria-label="Paper snapshot">
                    <div class="plan-c-snapshot-row plan-c-snapshot-head" role="row">
                      <span role="columnheader">Field</span>
                      <span role="columnheader">Value</span>
                    </div>
                    <div
                      v-for="row in selectedPlanCSnapshotRows"
                      :key="row.field"
                      class="plan-c-snapshot-row"
                      role="row"
                    >
                      <span class="plan-c-snapshot-field" role="cell">
                        <svg v-if="row.field === 'Methods'" viewBox="0 0 20 20" aria-hidden="true">
                          <path d="M6.5 4.5h7M5 8h10M6.5 11.5h7M8 15h4" />
                          <path d="M4.5 3.5h11v13h-11z" />
                        </svg>
                        <svg v-else-if="row.field === 'Outcomes'" viewBox="0 0 20 20" aria-hidden="true">
                          <path d="M10 3.5v13M5.2 7.5h9.6M6.2 7.5l-2.2 5h4.4zM13.8 7.5l-2.2 5h4.4z" />
                        </svg>
                        <svg v-else viewBox="0 0 20 20" aria-hidden="true">
                          <path d="M4.5 4.5h11v11h-11z" />
                          <path d="m6.8 10.4 2 2 4.4-5" />
                        </svg>
                        {{ row.field }}
                      </span>
                      <span class="plan-c-snapshot-value" role="cell">{{ row.value }}</span>
                    </div>
                  </div>
                </section>
                <section class="plan-c-detail-section plan-c-useful-section">
                  <h3>Why this source is useful</h3>
                  <p class="plan-c-useful-summary">{{ selectedPlanCSource.usefulSummary }}</p>
                </section>
                <section
                  class="plan-c-detail-section"
                  :class="{ 'plan-c-evidence-section': isPlanCAdvancedSelected }"
                >
                  <h3>{{ isPlanCAdvancedSelected ? 'Key Evidence (2)' : 'Key Findings (2)' }}</h3>
                  <template v-if="isPlanCAdvancedSelected">
                    <div
                      v-for="(finding, index) in selectedPlanCSource.findings"
                      :key="finding.text"
                      class="plan-c-evidence-card"
                    >
                      <span class="plan-c-evidence-icon" aria-hidden="true"></span>
                      <span class="plan-c-evidence-divider" aria-hidden="true"></span>
                      <p>
                        <strong>{{ index === 0 ? 'ABSTRACT' : 'INSTRUCTION' }}</strong>
                        <span>&middot;</span>
                        “{{ finding.quote }}”
                      </p>
                    </div>
                  </template>
                  <template v-else>
                    <div
                      v-for="(finding, index) in selectedPlanCSource.findings"
                      :key="finding.text"
                      class="plan-c-finding-card"
                    >
                      <p><strong>ABSTRACT</strong> &middot; “{{ finding.quote }}”</p>
                      <p><b>Finding {{ index + 1 }}</b> &middot; {{ finding.text }}</p>
                    </div>
                  </template>
                </section>
              </div>
            </section>
          </div>
        </template>

        <template v-else>
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
            <div v-if="!selectedPlanASource || isPlanBSelected" class="results-header">
              <template v-if="selectedProject === 'AI Research 方案B'">
                <h2>Research Result</h2>
                <div class="result-view-toggle" aria-label="Research result view">
                  <button
                    :class="{ active: selectedResultView === 'Result' }"
                    type="button"
                    @click="selectedResultView = 'Result'"
                  >
                    <svg viewBox="0 0 16 16" aria-hidden="true">
                      <path d="M2.5 4.5h.01M2.5 8h.01M2.5 11.5h.01M5 4.5h8M5 8h8M5 11.5h8" />
                    </svg>
                    Result
                  </button>
                  <button
                    :class="{ active: selectedResultView === 'Sources' }"
                    type="button"
                    @click="selectedResultView = 'Sources'"
                  >
                    <svg viewBox="0 0 16 16" aria-hidden="true">
                      <path d="M3 4h10M3 8h10M3 12h10" />
                      <path d="M3 3h10v10H3z" />
                    </svg>
                    Sources
                  </button>
                </div>
              </template>
              <template v-else>
                <h2><span>5</span> Sources Found</h2>
              </template>
              <div v-if="selectedProject !== 'AI Research 方案B'" class="result-actions">
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
                          :class="{ active: pendingResourceType === type }"
                          @click="pendingResourceType = type"
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
                      <button class="confirm-filter" type="button" @click="confirmFilters">
                        Confirm
                      </button>
                    </div>
                  </div>
                </div>
              </div>
            </div>

            <div v-if="isPlanBSelected && selectedResultView === 'Result'" class="research-result-report">
              <p>
                Educational data science in higher education is the use of data, statistics, and computational methods
                to improve teaching, learning, and institutional decision-making in universities. It typically includes
                learning analytics, educational data mining, and institutional analytics, all aimed at understanding
                student behavior, supporting success, and informing policy.
                <span
                  class="resource-citation"
                  :class="{ active: activeReferencePopoverId === 'summary-intro' }"
                  @mouseenter="showReferencePopover($event, 'summary-intro')"
                  @mouseleave="hideReferencePopoverSoon"
                >
                  <span
                    class="resource-tag"
                    @mouseenter="showReferencePopover($event, 'summary-intro')"
                  >
                    <span class="resource-tag-icon" aria-hidden="true"></span>
                    Eberle
                    <span>+1</span>
                  </span>
                  <span
                    class="resource-reference-popover"
                    :class="{ visible: activeReferencePopoverId === 'summary-intro' }"
                    role="tooltip"
                    :style="referencePopoverStyle"
                    @mouseenter="keepReferencePopoverOpen"
                    @mouseleave="hideReferencePopoverSoon"
                  >
                    <span class="reference-popover-header">
                      <span class="reference-pager"><button class="reference-nav-button" type="button" aria-label="Previous reference" @click.stop.prevent="showPreviousReference"><svg viewBox="0 0 20 20"><path d="m12 5-5 5 5 5" /></svg></button>{{ activeReferenceIndex + 1 }}/{{ referenceCards.length }}<button class="reference-nav-button" type="button" aria-label="Next reference" @click.stop.prevent="showNextReference"><svg viewBox="0 0 20 20"><path d="m8 5 5 5-5 5" /></svg></button></span>
                      <span>2 References</span>
                    </span>
                    <span class="reference-popover-card">
                      <strong class="reference-popover-title">{{ referenceCards[activeReferenceIndex].title }}</strong>
                      <span class="reference-quote">
                        <span class="reference-quote-icon" aria-hidden="true"></span>
                        <span>
                          <b>QUOTE</b>
                          <span>&middot;</span>
                          {{ referenceCards[activeReferenceIndex].quoteLead }}
                          {{ referenceCards[activeReferenceIndex].quoteBody }}
                        </span>
                      </span>
                      <span class="reference-meta">
                        <span><b>{{ referenceCards[activeReferenceIndex].year }}</b> &middot; {{ referenceCards[activeReferenceIndex].author }} &middot; {{ referenceCards[activeReferenceIndex].type }}</span><span><b>{{ referenceCards[activeReferenceIndex].citationCount }}</b> Citations</span>
                      </span>
                    </span>
                    <span class="reference-popover-actions">
                      <button
                        class="reference-action-button"
                        type="button"
                        aria-label="Cite reference"
                        @click.stop.prevent="openReferenceCitationDialog"
                      >
                        <span class="reference-action-icon cite"></span>
                      </button>
                      <span class="reference-action-icon save"></span>
                      <svg class="reference-action-svg" viewBox="0 0 20 20" aria-hidden="true"><path d="M8 5H5.5A1.5 1.5 0 0 0 4 6.5v8A1.5 1.5 0 0 0 5.5 16h8a1.5 1.5 0 0 0 1.5-1.5V12" /><path d="M11 4h5v5" /><path d="m10 10 6-6" /></svg>
                    </span>
                  </span>
                </span>
              </p>

              <section class="research-result-section">
                <h3>What it covers</h3>
                <ul>
                  <li>
                    Learning analytics: analyzing student activity in courses, often from LMS or digital platforms, to
                    identify engagement patterns and at-risk learners.
                    <span
                      class="resource-citation"
                      :class="{ active: activeReferencePopoverId === 'learning-analytics' }"
                      @mouseenter="showReferencePopover($event, 'learning-analytics')"
                      @mouseleave="hideReferencePopoverSoon"
                    >
                      <span
                        class="resource-tag"
                        @mouseenter="showReferencePopover($event, 'learning-analytics')"
                      >
                        <span class="resource-tag-icon" aria-hidden="true"></span>
                        Eberle
                        <span>+2</span>
                      </span>
                      <span
                        class="resource-reference-popover"
                        :class="{ visible: activeReferencePopoverId === 'learning-analytics' }"
                        role="tooltip"
                        :style="referencePopoverStyle"
                        @mouseenter="keepReferencePopoverOpen"
                        @mouseleave="hideReferencePopoverSoon"
                      >
                        <span class="reference-popover-header">
                          <span class="reference-pager"><button class="reference-nav-button" type="button" aria-label="Previous reference" @click.stop.prevent="showPreviousReference"><svg viewBox="0 0 20 20"><path d="m12 5-5 5 5 5" /></svg></button>{{ activeReferenceIndex + 1 }}/{{ referenceCards.length }}<button class="reference-nav-button" type="button" aria-label="Next reference" @click.stop.prevent="showNextReference"><svg viewBox="0 0 20 20"><path d="m8 5 5 5-5 5" /></svg></button></span>
                          <span>2 References</span>
                        </span>
                        <span class="reference-popover-card">
                          <strong class="reference-popover-title">{{ referenceCards[activeReferenceIndex].title }}</strong>
                          <span class="reference-quote">
                            <span class="reference-quote-icon" aria-hidden="true"></span>
                            <span>
                              <b>QUOTE</b>
                              <span>&middot;</span>
                              {{ referenceCards[activeReferenceIndex].quoteLead }}
                              {{ referenceCards[activeReferenceIndex].quoteBody }}
                            </span>
                          </span>
                          <span class="reference-meta">
                            <span><b>{{ referenceCards[activeReferenceIndex].year }}</b> &middot; {{ referenceCards[activeReferenceIndex].author }} &middot; {{ referenceCards[activeReferenceIndex].type }}</span><span><b>{{ referenceCards[activeReferenceIndex].citationCount }}</b> Citations</span>
                          </span>
                        </span>
                        <span class="reference-popover-actions">
                          <button
                            class="reference-action-button"
                            type="button"
                            aria-label="Cite reference"
                            @click.stop.prevent="openReferenceCitationDialog"
                          >
                            <span class="reference-action-icon cite"></span>
                          </button>
                          <span class="reference-action-icon save"></span>
                          <svg class="reference-action-svg" viewBox="0 0 20 20" aria-hidden="true"><path d="M8 5H5.5A1.5 1.5 0 0 0 4 6.5v8A1.5 1.5 0 0 0 5.5 16h8a1.5 1.5 0 0 0 1.5-1.5V12" /><path d="M11 4h5v5" /><path d="m10 10 6-6" /></svg>
                        </span>
                      </span>
                    </span>
                  </li>
                  <li>
                    Educational data mining: applying machine learning and statistical methods to educational datasets
                    to discover patterns and build predictive models.
                    <span
                      class="resource-citation"
                      :class="{ active: activeReferencePopoverId === 'data-mining' }"
                      @mouseenter="showReferencePopover($event, 'data-mining')"
                      @mouseleave="hideReferencePopoverSoon"
                    >
                      <span
                        class="resource-tag"
                        @mouseenter="showReferencePopover($event, 'data-mining')"
                      >
                        <span class="resource-tag-icon" aria-hidden="true"></span>
                        Eberle
                      </span>
                      <span
                        class="resource-reference-popover"
                        :class="{ visible: activeReferencePopoverId === 'data-mining' }"
                        role="tooltip"
                        :style="referencePopoverStyle"
                        @mouseenter="keepReferencePopoverOpen"
                        @mouseleave="hideReferencePopoverSoon"
                      >
                        <span class="reference-popover-header">
                          <span class="reference-pager"><button class="reference-nav-button" type="button" aria-label="Previous reference" @click.stop.prevent="showPreviousReference"><svg viewBox="0 0 20 20"><path d="m12 5-5 5 5 5" /></svg></button>{{ activeReferenceIndex + 1 }}/{{ referenceCards.length }}<button class="reference-nav-button" type="button" aria-label="Next reference" @click.stop.prevent="showNextReference"><svg viewBox="0 0 20 20"><path d="m8 5 5 5-5 5" /></svg></button></span>
                          <span>2 References</span>
                        </span>
                        <span class="reference-popover-card">
                          <strong class="reference-popover-title">{{ referenceCards[activeReferenceIndex].title }}</strong>
                          <span class="reference-quote">
                            <span class="reference-quote-icon" aria-hidden="true"></span>
                            <span>
                              <b>QUOTE</b>
                              <span>&middot;</span>
                              {{ referenceCards[activeReferenceIndex].quoteLead }}
                              {{ referenceCards[activeReferenceIndex].quoteBody }}
                            </span>
                          </span>
                          <span class="reference-meta">
                            <span><b>{{ referenceCards[activeReferenceIndex].year }}</b> &middot; {{ referenceCards[activeReferenceIndex].author }} &middot; {{ referenceCards[activeReferenceIndex].type }}</span><span><b>{{ referenceCards[activeReferenceIndex].citationCount }}</b> Citations</span>
                          </span>
                        </span>
                        <span class="reference-popover-actions">
                          <button
                            class="reference-action-button"
                            type="button"
                            aria-label="Cite reference"
                            @click.stop.prevent="openReferenceCitationDialog"
                          >
                            <span class="reference-action-icon cite"></span>
                          </button>
                          <span class="reference-action-icon save"></span>
                          <svg class="reference-action-svg" viewBox="0 0 20 20" aria-hidden="true"><path d="M8 5H5.5A1.5 1.5 0 0 0 4 6.5v8A1.5 1.5 0 0 0 5.5 16h8a1.5 1.5 0 0 0 1.5-1.5V12" /><path d="M11 4h5v5" /><path d="m10 10 6-6" /></svg>
                        </span>
                      </span>
                    </span>
                  </li>
                  <li>
                    Institutional analytics: using enrollment, progression, retention, and performance data to support
                    management and planning.
                  </li>
                  <li>
                    Research and practice: the field connects education theory with computer science and statistics to
                    improve educational decisions and outcomes.
                  </li>
                </ul>
              </section>
            </div>

            <div v-else-if="selectedPlanASource && !isPlanBSelected" class="plan-a-detail-page">
              <div class="plan-a-detail-nav">
                <button type="button" @click="closePlanASourceDetail">
                  <svg viewBox="0 0 20 20" aria-hidden="true">
                    <path d="m12 5-5 5 5 5" />
                  </svg>
                  Back to Source List
                </button>
              </div>

              <section class="plan-a-detail-hero">
                <h3>{{ selectedPlanASource.title }}</h3>
                <div class="plan-a-detail-meta">
                  <span>{{ selectedPlanASource.year }}</span>
                  <span>&middot;</span>
                  <span>{{ selectedPlanASource.author }} ... +6 more</span>
                  <span>&middot;</span>
                  <span>NeurIPS</span>
                  <span>&middot;</span>
                  <span>{{ selectedPlanASource.type }}</span>
                </div>
                <div class="plan-a-detail-actions">
                  <button type="button" @click="openCitationDialog(selectedPlanASource)">
                    <img :src="citeIcon" alt="" />
                    Cite
                  </button>
                  <button type="button">
                    <svg viewBox="0 0 20 20" aria-hidden="true">
                      <path d="M8.2 10.8 11.6 7.4a2.3 2.3 0 0 1 3.3 3.3l-4.4 4.4a3.2 3.2 0 0 1-4.5-4.5l5.2-5.2" />
                    </svg>
                    DOI
                  </button>
                  <button type="button">
                    <img :src="collectIcon" alt="" />
                    Save
                  </button>
                </div>
              </section>

              <section class="plan-a-detail-section">
                <h3>Why this source is useful</h3>
                <p>
                  This source is useful for {{ selectedPlanASource.bestUsedFor }}. It explains the core relationship
                  between behavioral prompts, motivation, and learner action, so it can support claims about where this
                  article fits in the background, framing, or argument section of the paper.
                </p>
              </section>

              <section class="plan-a-detail-section">
                <h3>Key Findings ({{ selectedPlanASource.quoteDetails.length }})</h3>
                <div class="plan-a-detail-findings">
                  <article
                    v-for="(quote, index) in selectedPlanASource.quoteDetails"
                    :key="`${selectedPlanASource.title}-${quote.label}-detail`"
                    class="plan-a-detail-finding"
                  >
                    <div class="plan-a-detail-quote">
                      <span class="plan-a-detail-quote-icon" aria-hidden="true"></span>
                      <span class="plan-a-detail-quote-rule" aria-hidden="true"></span>
                      <p>
                        <strong>Abstract</strong>
                        <span>&middot;</span>
                        “{{ quote.lead }} {{ quote.body }}”
                      </p>
                    </div>
                    <p class="plan-a-detail-finding-text">
                      <strong>Finding {{ index + 1 }}</strong>
                      <span>&middot;</span>
                      {{ selectedPlanASource.relevanceText || selectedPlanASource.takeaway }}
                    </p>
                  </article>
                </div>
              </section>

              <section class="plan-a-detail-section plan-a-detail-abstract">
                <h3>Abstract</h3>
                <p
                  class="plan-a-detail-abstract-text"
                  :class="{ collapsed: !isPlanAAbstractExpanded }"
                >
                  {{ selectedPlanASource.takeaway }}.
                  <template v-if="isPlanAAbstractExpanded">
                    {{ selectedPlanASource.suggestedSentence }}
                  </template>
                  <button type="button" @click="isPlanAAbstractExpanded = !isPlanAAbstractExpanded">
                    {{ isPlanAAbstractExpanded ? 'Show Less' : 'Show More' }}
                  </button>
                </p>
              </section>
            </div>

            <div v-else class="source-list">
              <article
                v-for="source in displayedSources"
                :key="getSourceKey(source)"
                class="source-card"
                :class="{
                  'source-card-plan-a': !isPlanBSelected,
                  'source-card-plan-b': isPlanBSelected,
                  'source-card-expanded': isQuoteExpanded(source),
                }"
                :tabindex="!isPlanBSelected ? 0 : undefined"
                :role="!isPlanBSelected ? 'button' : undefined"
                @click="openPlanASourceDetail(source)"
                @keydown.enter.prevent="openPlanASourceDetail(source)"
                @keydown.space.prevent="openPlanASourceDetail(source)"
              >
                <template v-if="!isPlanBSelected">
                  <div class="plan-a-card-main">
                    <h3>{{ source.title }}</h3>
                    <div class="plan-a-meta">
                      <strong>{{ source.year }}</strong>
                      <span>&middot;</span>
                      <span>{{ source.author }}</span>
                      <span>&middot;</span>
                      <span>{{ source.type }}</span>
                      <span>&middot;</span>
                      <span class="plan-a-citations">
                        <strong>{{ source.citationCount }}</strong>
                        Citations
                      </span>
                    </div>
                  </div>

                  <div class="plan-a-snapshot">
                    <p>
                      <strong>SNAPSHOT &middot;</strong>
                      {{ source.takeaway }}
                    </p>
                  </div>

                  <div class="plan-a-useful-pill">
                    <span class="plan-a-useful-icon" aria-hidden="true">&#128161;</span>
                    <span>This source is useful for {{ source.bestUsedFor }}</span>
                    <svg viewBox="0 0 20 20" aria-hidden="true">
                      <path d="M4 10h12" />
                      <path d="m11 5 5 5-5 5" />
                    </svg>
                  </div>
                </template>

                <template v-else>
                  <div class="source-top">
                    <h3>{{ source.title }}</h3>
                  <div
                    class="quote-wrapper"
                    :class="{
                      'quote-wrapper-plan-a': !isPlanBSelected,
                      'quote-wrapper-expanded': isQuoteExpanded(source),
                    }"
                  >
                    <button
                      v-if="!isQuoteExpanded(source)"
                      class="quote-pill"
                      :class="{ 'quote-pill-plan-b': isPlanBSelected }"
                      type="button"
                      aria-haspopup="dialog"
                      :aria-expanded="isQuoteExpanded(source)"
                      @click="toggleQuotePanel(source)"
                    >
                      <span class="quote-icon" aria-hidden="true"></span>
                      {{ source.quotes }}
                      <svg class="quote-chevron" viewBox="0 0 20 20" aria-hidden="true">
                        <path d="m6.5 8.2 3.5 3.5 3.5-3.5" />
                      </svg>
                    </button>
                    <div v-if="isQuoteExpanded(source)" class="quote-expanded-panel" role="region" aria-label="Relevant quotes">
                      <button class="quote-expanded-header" type="button" @click="toggleQuotePanel(source)">
                        <span>{{ source.quotes }}</span>
                        <svg viewBox="0 0 20 20" aria-hidden="true">
                          <path d="m6.5 11.8 3.5-3.5 3.5 3.5" />
                        </svg>
                      </button>
                      <div class="quote-expanded-list">
                        <div
                          v-for="quote in source.quoteDetails"
                          :key="`${source.title}-${quote.label}-expanded`"
                          class="quote-expanded-row"
                        >
                          <span class="quote-expanded-icon" aria-hidden="true"></span>
                          <div>
                            <p class="quote-expanded-lead">
                              <strong>{{ quote.label }}</strong>
                              <span>&middot;</span>
                              {{ quote.lead }}
                              {{ quote.body }}
                            </p>
                          </div>
                        </div>
                      </div>
                    </div>
                    <div v-if="!isPlanBSelected" class="quote-popover" role="dialog" aria-label="Relevant quotes">
                      <div v-for="quote in source.quoteDetails" :key="`${source.title}-${quote.label}`" class="quote-popover-row">
                        <span class="quote-popover-icon" aria-hidden="true"></span>
                        <p>
                          <strong>{{ quote.label }}</strong>
                          <span>&middot;</span>
                          {{ quote.lead }}
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
                      <template v-if="source.citationCount">
                        <span>&middot;</span>
                        <span class="source-citation-count">
                          <strong>{{ source.citationCount }}</strong>
                          Citations
                        </span>
                      </template>
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
                </template>
              </article>
            </div>
          </section>
        </div>
        </template>
      </section>

      <div class="debug-switcher" aria-label="System debug project switcher">
        <span>调试切换</span>
        <button
          v-for="option in debugProjectOptions"
          :key="option.project"
          type="button"
          :class="{ active: selectedProject === option.project }"
          @click="selectProject(option.project)"
        >
          {{ option.label }}
        </button>
      </div>
    </section>
    </main>

    <Teleport to="body">
      <span
        v-if="isPlanBSelected && selectedResultView === 'Result' && activeReferencePopoverId"
        class="resource-reference-popover reference-popover-portal visible"
        role="tooltip"
        :style="referencePopoverStyle"
        @mouseenter="keepReferencePopoverOpen"
        @mouseleave="hideReferencePopoverSoon"
      >
        <span class="reference-popover-header">
          <span class="reference-pager">
            <button
              class="reference-nav-button"
              type="button"
              aria-label="Previous reference"
              @click.stop.prevent="showPreviousReference"
            >
              <svg viewBox="0 0 20 20"><path d="m12 5-5 5 5 5" /></svg>
            </button>
            {{ activeReferenceIndex + 1 }}/{{ referenceCards.length }}
            <button
              class="reference-nav-button"
              type="button"
              aria-label="Next reference"
              @click.stop.prevent="showNextReference"
            >
              <svg viewBox="0 0 20 20"><path d="m8 5 5 5-5 5" /></svg>
            </button>
          </span>
          <span>2 References</span>
        </span>
        <span class="reference-popover-card">
          <strong class="reference-popover-title">{{ referenceCards[activeReferenceIndex].title }}</strong>
          <span class="reference-quote">
            <span class="reference-quote-icon" aria-hidden="true"></span>
            <span>
              <b>QUOTE</b>
              <span>&middot;</span>
              {{ referenceCards[activeReferenceIndex].quoteLead }}
              {{ referenceCards[activeReferenceIndex].quoteBody }}
            </span>
          </span>
          <span class="reference-meta">
            <span><b>{{ referenceCards[activeReferenceIndex].year }}</b> &middot; {{ referenceCards[activeReferenceIndex].author }} &middot; {{ referenceCards[activeReferenceIndex].type }}</span><span><b>{{ referenceCards[activeReferenceIndex].citationCount }}</b> Citations</span>
          </span>
        </span>
        <span class="reference-popover-actions">
          <button
            class="reference-action-button"
            type="button"
            aria-label="Cite reference"
            @click.stop.prevent="openReferenceCitationDialog"
          >
            <span class="reference-action-icon cite"></span>
          </button>
          <span class="reference-action-icon save"></span>
          <svg class="reference-action-svg" viewBox="0 0 20 20" aria-hidden="true"><path d="M8 5H5.5A1.5 1.5 0 0 0 4 6.5v8A1.5 1.5 0 0 0 5.5 16h8a1.5 1.5 0 0 0 1.5-1.5V12" /><path d="M11 4h5v5" /><path d="m10 10 6-6" /></svg>
        </span>
      </span>
    </Teleport>

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
