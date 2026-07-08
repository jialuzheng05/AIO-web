<script setup lang="ts">
import { computed, onBeforeUnmount, onMounted, ref } from 'vue'
import citeIcon from './assets/cite.svg'
import collectIcon from './assets/collect.svg'
import historyIcon from './assets/history_2.svg'
import listArrowIcon from './assets/list_arrow.svg'
import PlanC2Loading from './prototypes/PlanC2Loading.vue'
import PlanC3Loading from './prototypes/PlanC3Loading.vue'

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
const planCGenerationSteps = [
  'Understanding your research question',
  'Searching academic sources',
  'Evaluating credibility and relevance',
  'Preparing your results',
]

const projects = ['AI Research 方案C', 'AI Research 方案C-2', 'AI Research 方案C-3', 'AI Research', 'AI Research 方案B'] as const
type ProjectName = (typeof projects)[number]
const debugProjectOptions: Array<{ label: string; project: ProjectName }> = [
  { label: '方案C', project: 'AI Research 方案C' },
  { label: '方案C-2', project: 'AI Research 方案C-2' },
  { label: '方案C-3', project: 'AI Research 方案C-3' },
]
const selectedProject = ref<ProjectName>('AI Research 方案C')
const isPlanBSelected = computed(() => selectedProject.value === 'AI Research 方案B')
const isPlanC2Selected = computed(() => selectedProject.value === 'AI Research 方案C-2')
const isPlanC3Selected = computed(() => selectedProject.value === 'AI Research 方案C-3')
const isPlanCSelected = computed(() =>
  selectedProject.value === 'AI Research 方案C' ||
  selectedProject.value === 'AI Research 方案C-2' ||
  selectedProject.value === 'AI Research 方案C-3',
)
const planCQuery = ref('')
const hasPlanCQuery = computed(() => planCQuery.value.trim().length > 0)
const hasPlanCResults = ref(false)
const isPlanCGenerating = ref(false)
const isPlanCSearchActive = computed(() => hasPlanCResults.value || isPlanCGenerating.value)
const planCGeneratingStep = ref(0)
const selectedPlanCSourceIndex = ref(0)
const isPlanCSourceLoading = ref(false)
const isPlanCAbstractExpanded = ref(false)
const selectedResultView = ref<'Result' | 'Sources'>('Result')
const expandedQuoteSourceKey = ref<string | null>(null)
const expandedPlanASourceKey = ref<string | null>(null)
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
    usefulReasons: [
      'Defining educational data science within higher education contexts',
      'Explaining how learning analytics and data mining support student success',
      'Connecting institutional dashboards with teaching and advising decisions',
      'Discussing privacy, bias, and governance constraints for student data',
    ],
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
    usefulReasons: [
      'Describing dashboard use cases for higher education analytics',
      'Linking data visualizations to advising and teaching interventions',
      'Supporting claims about early-alert systems and student engagement',
      'Identifying limitations of risk scoring without contextual explanation',
    ],
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
    usefulReasons: [
      'Explaining prediction tasks in educational data science',
      'Comparing course-level and institution-level retention indicators',
      'Discussing how advisors act on risk predictions',
      'Highlighting interpretability concerns in student success models',
    ],
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
    usefulReasons: [
      'Framing privacy and consent issues in student data use',
      'Supporting arguments about responsible analytics governance',
      'Connecting model bias with institutional accountability',
      'Explaining why transparency matters for student trust',
    ],
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

const toggleProjectMenu = () => {
  isProjectMenuOpen.value = !isProjectMenuOpen.value
  isSortMenuOpen.value = false
  isFilterMenuOpen.value = false
}

const selectProject = (project: ProjectName) => {
  selectedProject.value = project
  isProjectMenuOpen.value = false

  if (project === 'AI Research 方案B') {
    selectedResultView.value = 'Result'
    expandedPlanASourceKey.value = null
  } else {
    expandedQuoteSourceKey.value = null
    expandedPlanASourceKey.value = null
  }
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
    planCSearchTimers = []
  }, stepDuration * planCGenerationSteps.length + 500)

  planCSearchTimers.push(finishTimer)
}

const selectPlanCSource = (index: number) => {
  if (index === selectedPlanCSourceIndex.value) return

  clearPlanCSourceLoadingTimer()
  selectedPlanCSourceIndex.value = index
  isPlanCSourceLoading.value = true
  isPlanCAbstractExpanded.value = false

  planCSourceLoadingTimer = window.setTimeout(() => {
    isPlanCSourceLoading.value = false
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
  planCGeneratingStep.value = 0
  selectedPlanCSourceIndex.value = 0
  isPlanCAbstractExpanded.value = false
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

      <section class="workspace-card" :class="{ 'workspace-card-plan-c': isPlanCSelected }" aria-label="AI Research">
        <template v-if="isPlanCSelected">
          <div class="plan-c-tabs" aria-label="AI Writing tool tabs">
            <button v-for="tab in tabs" :key="tab" :class="{ active: tab === 'AI Research' }" type="button">
              {{ tab }}
            </button>
          </div>

          <div class="plan-c-input-page">
            <section class="plan-c-input-pane">
              <div class="plan-c-input-header">
                <h2>{{ isPlanCGenerating ? 'Finding relevant sources...' : isPlanCSearchActive ? 'Your Search' : 'Find Sources for Your Paper' }}</h2>
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
                      @click="clearPlanCSearch"
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
                  <PlanC3Loading
                    v-if="isPlanC3Selected"
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
                  <div class="plan-c-results-bar">
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
                          <svg class="chevron" viewBox="0 0 20 20" aria-hidden="true">
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
                      class="plan-c-result-card"
                      :class="{ selected: selectedPlanCSourceIndex === index }"
                      @click="selectPlanCSource(index)"
                    >
                      <h3>{{ source.title }}</h3>
                      <div class="plan-c-result-tags">
                        <span v-for="tag in source.tags" :key="tag">{{ tag }}</span>
                      </div>
                      <p class="plan-c-snapshot"><strong>SNAPSHOT</strong> &middot; {{ source.snapshot }}</p>
                      <p class="plan-c-result-meta">
                        {{ source.year }} &middot; {{ source.author }} &middot; {{ source.venue }} &middot; {{ source.type }}
                      </p>
                    </article>
                  </div>
                </template>
              </template>
            </section>

            <section
              class="plan-c-empty-pane"
              :class="{ 'plan-c-details-pane': hasPlanCResults, 'plan-c-generating-pane': isPlanCGenerating }"
            >
              <div v-if="!isPlanCSearchActive" class="plan-c-empty-state">
                <div class="plan-c-empty-icon" aria-hidden="true">🔍</div>
                <div>
                  <h2>Search to Unlock Source Details</h2>
                  <p>Run a search on the left to explore abstracts, key findings, and useful metrics for your paper.</p>
                </div>
              </div>
              <div v-else-if="isPlanCGenerating && isPlanC3Selected" class="plan-c-skeleton-details" aria-hidden="true">
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
              <div v-else class="plan-c-source-details">
                <header>
                  <h2>Source Details</h2>
                </header>
                <section class="plan-c-detail-summary">
                  <h3>{{ selectedPlanCSource.title }}</h3>
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
                <section class="plan-c-detail-section">
                  <h3>Why this source is useful</h3>
                  <ul>
                    <li v-for="reason in selectedPlanCSource.usefulReasons" :key="reason">{{ reason }}</li>
                  </ul>
                </section>
                <section class="plan-c-detail-section">
                  <h3>Key Findings (2)</h3>
                  <div
                    v-for="(finding, index) in selectedPlanCSource.findings"
                    :key="finding.text"
                    class="plan-c-finding-card"
                  >
                    <p><strong>ABSTRACT</strong> &middot; “{{ finding.quote }}”</p>
                    <p><b>Finding {{ index + 1 }}</b> &middot; {{ finding.text }}</p>
                  </div>
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
            <div class="results-header">
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
              >
                <template v-if="!isPlanBSelected">
                  <div class="plan-a-card-actions" aria-label="Source actions">
                    <button
                      class="plan-a-cite-action"
                      type="button"
                      aria-label="Cite source"
                      @click.stop="openCitationDialog(source)"
                    >
                      <img :src="citeIcon" alt="" />
                      <span>Cite</span>
                    </button>
                    <button class="plan-a-save-action" type="button" aria-label="Save source">
                      <img :src="collectIcon" alt="" />
                    </button>
                  </div>
                  <div class="plan-a-card-main">
                    <h3>{{ source.title }}</h3>
                    <div class="plan-a-meta">
                      <strong>{{ source.year }}</strong>
                      <span>&middot;</span>
                      <span class="plan-a-citations">
                        <strong>{{ source.citationCount }}</strong>
                        citations
                      </span>
                      <span>&middot;</span>
                      <span>{{ source.author }}</span>
                      <span>&middot;</span>
                      <span class="plan-a-type">
                        <svg viewBox="0 0 16 16" aria-hidden="true">
                          <path d="M3.5 2.8h3.4c.6 0 1.1.5 1.1 1.1v9.3c0-.6-.5-1.1-1.1-1.1H3.5zM8 3.9c0-.6.5-1.1 1.1-1.1h3.4v9.3H9.1c-.6 0-1.1.5-1.1 1.1" />
                        </svg>
                        {{ source.type }}
                      </span>
                    </div>
                  </div>

                  <div class="plan-a-relevance">
                    <strong>{{ source.relevanceLabel }}</strong>
                    <span>&middot;</span>
                    <span>{{ source.relevanceText }}</span>
                  </div>

                  <div class="plan-a-best-used">
                    <strong>Best used for:</strong>
                    <span>{{ source.bestUsedFor }}</span>
                  </div>

                  <section class="suggested-sentence">
                    <div class="suggested-sentence-header">
                      <h4>Suggested sentence</h4>
                      <button
                        type="button"
                        aria-label="Copy suggested sentence"
                        @click.stop="copySuggestedSentence(source)"
                      >
                        <svg viewBox="0 0 20 20" aria-hidden="true">
                          <path d="M7 7.5h8v9H7z" />
                          <path d="M5 13.5H4.5A1.5 1.5 0 0 1 3 12V4.5A1.5 1.5 0 0 1 4.5 3H12a1.5 1.5 0 0 1 1.5 1.5V5" />
                        </svg>
                      </button>
                    </div>
                    <p>{{ source.suggestedSentence }}</p>
                  </section>

                  <div v-if="isPlanAExpanded(source)" class="plan-a-expanded-content">
                    <button
                      class="other-ways-header"
                      type="button"
                      :aria-expanded="isPlanAExpanded(source)"
                      @click="togglePlanAExpansion(source)"
                    >
                      <span>Other ways to use:</span>
                      <svg viewBox="0 0 20 20" aria-hidden="true">
                        <path d="m6.5 11.8 3.5-3.5 3.5 3.5" />
                      </svg>
                    </button>
                    <section
                      v-for="way in source.otherWays"
                      :key="`${source.title}-${way.title}`"
                      class="suggested-sentence other-way-card"
                    >
                      <div class="suggested-sentence-header">
                        <h4>{{ way.title }}</h4>
                        <button
                          type="button"
                          :aria-label="`Copy ${way.title}`"
                          @click.stop="copyPlainText(way.sentence)"
                        >
                          <svg viewBox="0 0 20 20" aria-hidden="true">
                            <path d="M7 7.5h8v9H7z" />
                            <path d="M5 13.5H4.5A1.5 1.5 0 0 1 3 12V4.5A1.5 1.5 0 0 1 4.5 3H12a1.5 1.5 0 0 1 1.5 1.5V5" />
                          </svg>
                        </button>
                      </div>
                      <p>{{ way.sentence }}</p>
                    </section>
                  </div>

                  <button
                    v-else
                    class="show-other-ways"
                    type="button"
                    :aria-expanded="isPlanAExpanded(source)"
                    @click="togglePlanAExpansion(source)"
                  >
                    <span>Show other ways to use</span>
                    <svg viewBox="0 0 20 20" aria-hidden="true">
                      <path d="m6.5 8.2 3.5 3.5 3.5-3.5" />
                    </svg>
                  </button>
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
