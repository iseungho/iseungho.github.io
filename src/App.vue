<script setup lang="ts">
import { computed, nextTick, onMounted, onUnmounted, reactive, ref } from 'vue'

const projectLinks = {
  frontStars: 'https://github.com/iseungho/Front-STARS',
  frontStarsVideo: 'https://youtu.be/_UF1LQTPCyA',
  capeasy: 'https://github.com/iseungho/Capeasy',
  capeasyVideo: 'https://youtu.be/1_jV6IfomLs?si=04fPvdS1_v__2jb2',
  researchPaper: 'https://drive.google.com/file/d/1BSQGQ4Va-bEbjrWlNvYnOXFJJNKfgDvM/view?usp=sharing',
  worldCloud: 'https://github.com/iseungho/Project-WorldCloud',
  pangyeori: 'https://github.com/pangyeori/pangyeori-fe',
}

const isDark = ref(false)
const activeSection = ref('top')
const imageDialog = ref<HTMLDialogElement | null>(null)
let swipeStartX = 0
let swipeStartY = 0
let suppressImageOpenUntil = 0
let mouseDragStartX = 0
let mouseDragStartY = 0
let mouseDragging = false

const frontStarsSlides = [
  { src: '/projects/front-stars/01-user-main.png', alt: '서울시 관광지 혼잡도와 사고 알림을 표시한 사용자 지도 화면', label: '사용자 메인 페이지' },
  { src: '/projects/front-stars/02-place-summary.png', alt: '관광지 인구와 날씨 및 주변 시설을 요약한 화면', label: '사용자 장소 요약' },
  { src: '/projects/front-stars/03-nearby-business.png', alt: '관광지 주변 상권 목록과 장소 정보를 지도에 표시한 화면', label: '사용자 주변 상권' },
  { src: '/projects/front-stars/04-mypage.png', alt: '회원 정보와 계정 관리 기능을 제공하는 마이페이지', label: '사용자 마이페이지' },
  { src: '/projects/front-stars/05-favorites.png', alt: '저장한 장소를 검색하고 분류하는 즐겨찾기 화면', label: '사용자 마이페이지 · 즐겨찾기' },
  { src: '/projects/front-stars/06-admin-dashboard.png', alt: '관광지 인구와 혼잡도 및 방문객 통계를 시각화한 관리자 화면', label: '관리자 대시보드' },
  { src: '/projects/front-stars/07-admin-traffic.png', alt: '도로 소통 상태와 주차 정보를 지도에 표시한 관리자 화면', label: '관리자 교통 · 주차 현황' },
]

const capeasySlides = [
  { src: '/projects/capeasy/01-main.png', alt: 'Capeasy 데스크톱과 모바일 메인 페이지', label: '메인 페이지' },
  { src: '/projects/capeasy/02-auth.png', alt: 'Capeasy 회원가입과 로그인 화면', label: '회원가입 · 로그인' },
  { src: '/projects/capeasy/03-upload.png', alt: 'Capeasy 공간 이미지 생성을 위한 동영상 입력 화면', label: '동영상 입력' },
  { src: '/projects/capeasy/04-result.png', alt: 'Capeasy 공간 이미지 생성 결과와 게시글 작성 화면', label: '생성 결과 · 게시글 작성' },
  { src: '/projects/capeasy/05-community.png', alt: 'Capeasy 게시글 목록과 상세 화면', label: '커뮤니티' },
]

const worldCloudSlides = [
  { src: '/projects/worldcloud-select.png', alt: 'WorldCloud 대주제 선택 화면', label: '주제 선택' },
  { src: '/projects/worldcloud-result.png', alt: 'WorldCloud 뉴스 분석 결과 화면', label: '분석 결과' },
  { src: '/projects/worldcloud-keywords.png', alt: 'LDA 토픽 분포와 주요 키워드를 시각화한 분석 결과', label: '키워드 분석 결과' },
]

const pangyeoriSlides = [
  { src: '/projects/pangyeori/toy-project.png', alt: 'AI가 판정하는 1대1 토론 서비스 판겨리 소개 화면', label: '판겨리 Toy Project' },
]

const activitySlides = [
  { src: '/activities/01.png', alt: '미래내일 일경험 및 뉴노멀 기업 연계 프로젝트 수료', label: '기업 연계 프로젝트', title: '미래내일 일경험 · 뉴노멀 프로젝트', meta: '위니드소프트 디지털 트윈 플랫폼 개발 참여', projectHref: null, projectLabel: null, resourceHref: null, resourceLabel: null },
  { src: '/activities/02.png', alt: '2024 세종 DX 해커톤 우수상 수상', label: '세종 DX 해커톤', title: '2024 SW융합클러스터 세종 DX 해커톤', meta: '우수상 · 2024.08', projectHref: null, projectLabel: null, resourceHref: null, resourceLabel: null },
  { src: '/activities/03.png', alt: '제21회 소프트웨어융합학과 학술제 대상 수상', label: '학술제', title: '제21회 소프트웨어융합학과 학술제', meta: '대상 · 2024.11', projectHref: '#capeasy', projectLabel: 'Capeasy 프로젝트 보기', resourceHref: null, resourceLabel: null },
  { src: '/activities/04.png', alt: '대한전자공학회 2024년도 추계학술대회 논문 제출', label: '학술대회', title: '대한전자공학회 2024 추계학술대회', meta: '학부생 논문경진대회 멀티미디어 분야 논문 제출', projectHref: '#capeasy', projectLabel: 'Capeasy 프로젝트 보기', resourceHref: projectLinks.researchPaper, resourceLabel: '논문 원문 보기' },
  { src: '/activities/06-paper.png', alt: '초해상화 기반 파노라마 스티칭 시스템 왜곡 최소화 연구 논문 첫 페이지', label: '학술대회 논문', title: '초해상화 기반 파노라마 스티칭 시스템에서 왜곡 현상 최소화 연구', meta: '대한전자공학회 2024 추계학술대회 · 학부생 논문경진대회', projectHref: '#capeasy', projectLabel: 'Capeasy 프로젝트 보기', resourceHref: projectLinks.researchPaper, resourceLabel: '논문 원문 보기' },
  { src: '/activities/05.png', alt: '하이세종페스티벌 프로젝트 전시 참여', label: '프로젝트 전시', title: 'HI-SEJONG FESTIVAL', meta: '프로젝트 전시 참여 · 2024.12', projectHref: '#capeasy', projectLabel: 'Capeasy 프로젝트 보기', resourceHref: null, resourceLabel: null },
]

const gallerySlides = { frontStars: frontStarsSlides, capeasy: capeasySlides, worldCloud: worldCloudSlides, pangyeori: pangyeoriSlides, activities: activitySlides }
type GalleryName = keyof typeof gallerySlides

const slideIndexes = reactive<Record<GalleryName, number>>({ frontStars: 0, capeasy: 0, worldCloud: 0, pangyeori: 0, activities: 0 })
const hintVisible = reactive<Record<GalleryName, boolean>>({ frontStars: false, capeasy: false, worldCloud: false, pangyeori: false, activities: false })
const modalGallery = ref<GalleryName | null>(null)
const modalSlide = ref(0)
const activeModalSlide = computed(() => modalGallery.value ? gallerySlides[modalGallery.value][modalSlide.value] : null)
const activeModalCaption = computed(() => {
  const slide = activeModalSlide.value
  return slide && 'title' in slide ? `${slide.label} · ${slide.title}` : slide?.label
})
let hintObserver: IntersectionObserver | undefined
let sectionObserver: IntersectionObserver | undefined
let updateOutlineContrast: (() => void) | undefined
const hintTimers: number[] = []

const pageSections = [
  { id: 'top', label: '소개' },
  { id: 'projects', label: '프로젝트' },
  { id: 'pangyeori', label: 'Toy Project' },
  { id: 'about', label: '역량' },
  { id: 'activities', label: '대외 활동' },
  { id: 'experience', label: '경력' },
  { id: 'education', label: '교육' },
  { id: 'credentials', label: '자격 · 수상' },
]

const moveSlide = (gallery: GalleryName, direction: number) => {
  const length = gallerySlides[gallery].length
  slideIndexes[gallery] = (slideIndexes[gallery] + direction + length) % length
}

const startSwipe = (event: TouchEvent) => {
  swipeStartX = event.changedTouches[0].clientX
  swipeStartY = event.changedTouches[0].clientY
}
const swipeDirection = (event: TouchEvent) => {
  const distanceX = swipeStartX - event.changedTouches[0].clientX
  const distanceY = swipeStartY - event.changedTouches[0].clientY
  return Math.abs(distanceX) > 45 && Math.abs(distanceX) > Math.abs(distanceY) * 1.2 ? (distanceX > 0 ? 1 : -1) : 0
}
const endSwipe = (gallery: GalleryName, event: TouchEvent) => {
  const direction = swipeDirection(event)
  if (direction) {
    suppressImageOpenUntil = performance.now() + 350
    moveSlide(gallery, direction)
  }
}

const startMouseDrag = (event: PointerEvent) => {
  if (event.pointerType !== 'mouse' || event.button !== 0) return
  mouseDragging = true
  mouseDragStartX = event.clientX
  mouseDragStartY = event.clientY
}
const endMouseDrag = (gallery: GalleryName, event: PointerEvent) => {
  if (!mouseDragging || event.pointerType !== 'mouse') return
  mouseDragging = false
  const distanceX = mouseDragStartX - event.clientX
  const distanceY = mouseDragStartY - event.clientY
  if (Math.abs(distanceX) > 45 && Math.abs(distanceX) > Math.abs(distanceY) * 1.2) {
    suppressImageOpenUntil = performance.now() + 350
    moveSlide(gallery, distanceX > 0 ? 1 : -1)
  }
}
const cancelMouseDrag = () => { mouseDragging = false }

const openImage = async (gallery: GalleryName, index: number) => {
  if (performance.now() < suppressImageOpenUntil) return
  modalGallery.value = gallery
  modalSlide.value = index
  await nextTick()
  imageDialog.value?.showModal()
}
const closeImage = () => imageDialog.value?.close()
const moveModal = (direction: number) => {
  if (!modalGallery.value) return
  const length = gallerySlides[modalGallery.value].length
  modalSlide.value = (modalSlide.value + direction + length) % length
}
const endModalSwipe = (event: TouchEvent) => {
  const direction = swipeDirection(event)
  if (direction) moveModal(direction)
}

const applyTheme = (dark: boolean) => {
  isDark.value = dark
  document.documentElement.classList.toggle('dark', dark)
  document.documentElement.style.colorScheme = dark ? 'dark' : 'light'
}

const toggleTheme = () => {
  applyTheme(!isDark.value)
  localStorage.setItem('portfolio-theme', isDark.value ? 'dark' : 'light')
}

onMounted(() => {
  const savedTheme = localStorage.getItem('portfolio-theme')
  const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches
  applyTheme(savedTheme ? savedTheme === 'dark' : prefersDark)

  hintObserver = new IntersectionObserver((entries) => {
    entries.forEach((entry) => {
      if (!entry.isIntersecting) return
      const gallery = (entry.target as HTMLElement).dataset.gallery as GalleryName
      hintVisible[gallery] = true
      hintTimers.push(window.setTimeout(() => { hintVisible[gallery] = false }, 2200))
      hintObserver?.unobserve(entry.target)
    })
  }, { threshold: .35 })
  document.querySelectorAll<HTMLElement>('[data-gallery]').forEach((element) => hintObserver?.observe(element))

  sectionObserver = new IntersectionObserver((entries) => {
    const visibleSection = entries.find((entry) => entry.isIntersecting)
    if (visibleSection) activeSection.value = visibleSection.target.id
  }, { rootMargin: '-18% 0px -70% 0px' })
  pageSections.forEach(({ id }) => {
    const section = document.getElementById(id)
    if (section) sectionObserver?.observe(section)
  })

  const accentSection = document.querySelector<HTMLElement>('.now-section')
  const outlineItems = document.querySelectorAll<HTMLElement>('.page-outline > p, .page-outline a')
  if (accentSection) {
    updateOutlineContrast = () => {
      const accentRect = accentSection.getBoundingClientRect()
      outlineItems.forEach((item) => {
        const itemRect = item.getBoundingClientRect()
        const overlapTop = Math.max(itemRect.top, accentRect.top)
        const overlapBottom = Math.min(itemRect.bottom, accentRect.bottom)
        const hasOverlap = overlapBottom > overlapTop

        item.classList.toggle('accent-overlap', hasOverlap)
        if (hasOverlap) {
          const from = ((overlapTop - itemRect.top) / itemRect.height) * 100
          const to = ((overlapBottom - itemRect.top) / itemRect.height) * 100
          item.style.setProperty('--accent-from', `${Math.max(0, from)}%`)
          item.style.setProperty('--accent-to', `${Math.min(100, to)}%`)
        } else {
          item.style.removeProperty('--accent-from')
          item.style.removeProperty('--accent-to')
        }
      })
    }
    window.addEventListener('scroll', updateOutlineContrast, { passive: true })
    window.addEventListener('resize', updateOutlineContrast)
    updateOutlineContrast()
  }
})

onUnmounted(() => {
  hintObserver?.disconnect()
  sectionObserver?.disconnect()
  if (updateOutlineContrast) {
    window.removeEventListener('scroll', updateOutlineContrast)
    window.removeEventListener('resize', updateOutlineContrast)
  }
  hintTimers.forEach((timer) => window.clearTimeout(timer))
})

const skills = [
  { index: '01', title: '화면 구조 설계', description: '사용자 흐름을 먼저 정리하고, 재사용 가능한 컴포넌트와 명확한 상태 구조로 화면을 설계합니다.', keywords: ['Vue 3', 'React', 'TypeScript', 'Component Design'] },
  { index: '02', title: '데이터 시각화', description: '지도, 차트, 실시간 상태처럼 복잡한 데이터를 빠르게 읽고 판단할 수 있는 정보로 바꿉니다.', keywords: ['Mapbox GL', 'Recharts', 'Public API', 'Real-time UI'] },
  { index: '03', title: '연결과 완성', description: '프론트엔드에 머무르지 않고 API 연동, 배포 환경, 협업 흐름까지 이해하며 제품을 완성합니다.', keywords: ['Vite', 'REST API', 'AWS EC2', 'GitHub Actions'] },
]

const careers = [
  { period: '2025.09 — 현재', company: 'SK쉴더스', role: '보안관제', description: 'SK하이닉스 및 인천국제공항공사 보안관제 업무에서 침해징후 탐지, 이상 행위 모니터링과 위협 초동 대응을 수행합니다. 운영 현장에서 실시간 정보를 분석하고 우선순위를 판단한 경험을, 복잡한 데이터를 명확한 화면으로 전달하는 프론트엔드 개발 역량으로 연결하고 있습니다.' },
]

const awards = [
  { year: '2024.11', title: '소프트웨어융합학과 학술제 대상', detail: '홍익대학교' },
  { year: '2024.11', title: 'StoryLab Best Content상', detail: '뉴노멀 프로젝트' },
  { year: '2024.08', title: 'SW융합클러스터 해커톤 우수상', detail: '세종 DX 해커톤' },
  { year: '2023.09', title: 'SW융합클러스터 해커톤 인재상', detail: '디지털 콘텐츠 DX 해커톤' },
]
</script>

<template>
  <div class="site-shell">
    <a href="#main" class="skip-link">본문으로 바로가기</a>

    <header class="site-header">
      <nav class="nav-wrap" aria-label="주요 메뉴">
        <a href="#top" class="brand" aria-label="이승호 포트폴리오 홈">SEUNGHO.LEE</a>
        <div class="nav-links">
          <a href="#projects">프로젝트</a>
          <a href="#about">역량</a>
          <a href="#activities">활동</a>
          <a href="#experience">경력</a>
          <a href="#education">교육</a>
          <a href="#credentials">자격 · 수상</a>
        </div>
      </nav>
    </header>

    <nav class="floating-actions" aria-label="화면 설정">
      <button class="floating-button" type="button" :aria-label="isDark ? '라이트 모드로 전환' : '다크 모드로 전환'" :aria-pressed="isDark" :title="isDark ? '라이트 모드' : '다크 모드'" @click="toggleTheme">
        <svg v-if="isDark" viewBox="0 0 24 24" aria-hidden="true"><circle cx="12" cy="12" r="4"/><path d="M12 2v2M12 20v2M4.93 4.93l1.42 1.42M17.66 17.66l1.41 1.41M2 12h2M20 12h2M4.93 19.07l1.42-1.42M17.66 6.34l1.41-1.41"/></svg>
        <svg v-else viewBox="0 0 24 24" aria-hidden="true"><path d="M21 12.8A9 9 0 1 1 11.2 3 7 7 0 0 0 21 12.8Z"/></svg>
      </button>
      <a href="#top" class="floating-button" aria-label="맨 위로 이동" title="맨 위로">
        <svg viewBox="0 0 24 24" aria-hidden="true"><path d="m6 15 6-6 6 6"/></svg>
      </a>
    </nav>

    <aside class="page-outline" aria-label="포트폴리오 목차">
      <p><span>Portfolio</span><span class="outline-accent" aria-hidden="true">Portfolio</span></p>
      <nav>
        <a v-for="section in pageSections" :key="section.id" :href="`#${section.id}`" :class="{ active: activeSection === section.id }" :aria-current="activeSection === section.id ? 'location' : undefined">
          <span>{{ section.label }}</span><span class="outline-accent" aria-hidden="true">{{ section.label }}</span>
        </a>
      </nav>
    </aside>

    <main id="main">
      <section id="top" class="hero grid-backdrop">
        <div class="content hero-grid">
          <div>
            <p class="eyebrow"><span class="status-dot"></span>Frontend Developer</p>
            <h1>복잡한 데이터를<br><span>명확한 화면</span>으로<br>바꿉니다.</h1>
            <p class="hero-copy">지도·실시간 데이터·미디어 처리처럼 복잡한 흐름을 사용자가 바로 이해하고 행동할 수 있는 인터페이스로 설계하는 프론트엔드 개발자 이승호입니다.</p>
            <div class="button-row">
              <a href="#projects" class="button button-primary">대표 프로젝트 보기 <span aria-hidden="true">↓</span></a>
              <a href="mailto:ghehf51@naver.com" class="button button-secondary"><svg viewBox="0 0 24 24" aria-hidden="true" focusable="false"><path d="M3 3h5.2l7.6 10.9V3H21v18h-5.2L8.2 10.1V21H3V3Z"/></svg>이메일 보내기</a>
            </div>
          </div>

          <aside class="profile-card" aria-label="프로필 요약">
            <p class="profile-label">PROFILE / 2026</p>
            <p class="profile-name">이승호</p>
            <p class="profile-role">Frontend Developer</p>
            <dl class="profile-facts">
              <div><dt>Focus</dt><dd>UI Architecture</dd></div>
              <div><dt>Experience</dt><dd>Team Lead</dd></div>
              <div><dt>Strength</dt><dd>Data Visualization</dd></div>
              <div><dt>Stack</dt><dd>Vue · React · TS</dd></div>
            </dl>
          </aside>
        </div>
      </section>

      <section id="projects" class="section">
        <div class="content">
          <header class="section-heading">
            <div><p class="eyebrow">Selected Work</p><h2>대표 프로젝트</h2></div>
            <p>문제를 발견하고, 화면 구조를 설계하고, 실제 서비스 흐름으로 연결한 경험입니다.</p>
          </header>

          <div class="project-stack">
            <article id="sentry" class="case-study detailed-case">
              <div class="case-copy">
                <div class="case-meta"><span>01</span><p>2025.03 — 2025.06 · Frontend Lead</p></div>
                <h3>서울시 관광지 혼잡도 관제 시스템</h3>
                <p class="case-lead">월평균 약 4,800만 명이 찾는 서울 관광지의 인구 밀집도·기상·도로·주차 데이터를 분석해, 관광객과 행정 담당자가 혼잡 상황을 빠르게 판단하도록 만든 실시간 관제 서비스입니다.</p>
                <dl class="case-details">
                  <div><dt>배경</dt><dd>월평균 35.3건의 문화행사와 약 20건의 프로 스포츠 경기로 특정 지역에 관광객이 집중되지만, 분산된 정보만으로 한산한 장소와 위험 상황을 즉시 파악하기 어려웠습니다.</dd></div>
                  <div><dt>사용자</dt><dd>행정 담당자에게는 실시간 정책·관제 도구를, 관광객에게는 덜 붐비는 장소와 취향 기반 추천 정보를 제공하는 흐름으로 구분했습니다.</dd></div>
                  <div><dt>역할</dt><dd>Frontend Lead로 개발 환경과 공통 컴포넌트 구조를 설계하고, 도메인별 화면 구현 순서와 API 연동 기준을 정리했습니다.</dd></div>
                  <div><dt>설계</dt><dd>React·Vite·TypeScript SPA에 Redux Toolkit 상태 관리와 React Router 라우팅을 적용하고, 훅·유틸·스타일을 공통 영역으로 분리했습니다.</dd></div>
                  <div><dt>구현</dt><dd>REST API·JWT 인증을 연결해 Mapbox 지도, 실시간 혼잡·사건 알림, 장소 추천, 주변 상권, 즐겨찾기와 관리자 통계·교통 화면을 구현했습니다.</dd></div>
                </dl>
                <div class="tag-list"><span v-for="item in ['React', 'TypeScript', 'Redux Toolkit', 'Mapbox GL', 'Recharts', 'Tailwind CSS']" :key="item">{{ item }}</span></div>
                <div class="project-link-row">
                  <a :href="projectLinks.frontStars" target="_blank" rel="noreferrer" class="text-link"><svg viewBox="0 0 24 24" aria-hidden="true" focusable="false"><path fill-rule="evenodd" d="M12 2C6.48 2 2 6.58 2 12.23c0 4.52 2.87 8.35 6.84 9.71.5.1.68-.22.68-.49 0-.24-.01-1.04-.01-1.89-2.78.62-3.37-1.21-3.37-1.21-.45-1.18-1.11-1.49-1.11-1.49-.91-.64.07-.62.07-.62 1 .07 1.53 1.05 1.53 1.05.89 1.57 2.34 1.11 2.91.85.09-.66.35-1.11.63-1.36-2.22-.26-4.56-1.14-4.56-5.06 0-1.12.39-2.03 1.03-2.75-.1-.26-.45-1.3.1-2.71 0 0 .84-.28 2.75 1.05A9.3 9.3 0 0 1 12 6.97a9.3 9.3 0 0 1 2.5.35c1.91-1.33 2.75-1.05 2.75-1.05.55 1.41.2 2.45.1 2.71.64.72 1.03 1.63 1.03 2.75 0 3.93-2.34 4.79-4.57 5.05.36.32.68.94.68 1.9 0 1.37-.01 2.47-.01 2.81 0 .27.18.59.69.49A10.25 10.25 0 0 0 22 12.23C22 6.58 17.52 2 12 2Z" clip-rule="evenodd"/></svg>GitHub에서 자세히 보기 ↗</a>
                  <a :href="projectLinks.frontStarsVideo" target="_blank" rel="noreferrer" class="text-link"><svg viewBox="0 0 24 24" aria-hidden="true" focusable="false"><path d="M23.5 6.2a3 3 0 0 0-2.1-2.1C19.5 3.6 12 3.6 12 3.6s-7.5 0-9.4.5A3 3 0 0 0 .5 6.2 31 31 0 0 0 0 12a31 31 0 0 0 .5 5.8 3 3 0 0 0 2.1 2.1c1.9.5 9.4.5 9.4.5s7.5 0 9.4-.5a3 3 0 0 0 2.1-2.1A31 31 0 0 0 24 12a31 31 0 0 0-.5-5.8ZM9.6 15.6V8.4l6.3 3.6-6.3 3.6Z"/></svg>시연 영상 보기 ↗</a>
                </div>
              </div>
              <div class="case-media-stack">
                <div class="inline-video">
                  <iframe src="https://www.youtube-nocookie.com/embed/_UF1LQTPCyA" title="서울시 관광지 혼잡도 관제 시스템 시연 영상" loading="lazy" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
                  <div class="media-caption"><span>PROJECT DEMO</span><p>실시간 지도와 관제 흐름을 영상으로 확인할 수 있습니다.</p></div>
                </div>
                <div class="gallery project-gallery" :class="{ 'show-nav': hintVisible.frontStars }" data-gallery="frontStars" role="region" aria-roledescription="carousel" aria-label="Sentry 주요 화면" tabindex="0" @keydown.left.prevent="moveSlide('frontStars', -1)" @keydown.right.prevent="moveSlide('frontStars', 1)" @touchstart.passive="startSwipe" @touchend.passive="endSwipe('frontStars', $event)" @pointerdown="startMouseDrag" @pointerup="endMouseDrag('frontStars', $event)" @pointercancel="cancelMouseDrag">
                  <div class="slide-counter" aria-live="polite">{{ slideIndexes.frontStars + 1 }} / {{ frontStarsSlides.length }}</div>
                  <div class="carousel-viewport">
                    <div class="carousel-track" :style="{ transform: `translateX(-${slideIndexes.frontStars * 100}%)` }">
                      <figure v-for="(slide, index) in frontStarsSlides" :key="slide.src" :aria-hidden="index !== slideIndexes.frontStars">
                        <button class="image-open-button" type="button" :aria-label="`${slide.label} 크게 보기`" :tabindex="index === slideIndexes.frontStars ? 0 : -1" @click="openImage('frontStars', index)">
                          <img :src="slide.src" :alt="slide.alt" loading="lazy" draggable="false">
                        </button>
                        <figcaption>{{ slide.label }}</figcaption>
                      </figure>
                    </div>
                  </div>
                  <button type="button" class="slide-button slide-prev" aria-label="Sentry 이전 이미지" @click="moveSlide('frontStars', -1)"></button>
                  <button type="button" class="slide-button slide-next" aria-label="Sentry 다음 이미지" @click="moveSlide('frontStars', 1)"></button>
                  <p v-if="hintVisible.frontStars" class="swipe-hint" aria-hidden="true">↔ 좌우로 밀어보기</p>
                </div>
              </div>
            </article>

            <article id="capeasy" class="case-study detailed-case">
              <div class="case-copy">
                <div class="case-meta"><span>02</span><p>2024.03 — 2024.11 · Frontend</p></div>
                <h3>Capeasy</h3>
                <p class="case-kicker">동영상 기반 360도 공간 이미지 제작·관리 시스템</p>
                <p class="case-lead">동영상 입력부터 변환 대기, 360도 결과 확인·다운로드, 게시글 공유까지 분리된 기능을 하나의 사용자 흐름으로 연결했습니다.</p>
                <dl class="case-details">
                  <div><dt>문제</dt><dd>영상 업로드 후 별도의 변환 시간이 필요한 서비스에서 사용자가 현재 처리 단계와 다음 행동을 놓치지 않도록 흐름을 설계해야 했습니다.</dd></div>
                  <div><dt>역할</dt><dd>React·Redux 기반 프론트엔드 전반을 담당해 인증, 공간 이미지 제작, 결과 활용, 커뮤니티와 마이페이지 화면을 구현했습니다.</dd></div>
                  <div><dt>업로드</dt><dd>클릭·드래그앤드롭 입력, video MIME 검증, Object URL 기반 미리보기와 메모리 해제, 로딩·오류 피드백을 구성했습니다.</dd></div>
                  <div><dt>결과</dt><dd>Base64 응답을 Blob URL로 변환하고 VIEW360의 EquirectProjection으로 렌더링해, 전체 화면 확인·다운로드·게시글 작성으로 연결했습니다.</dd></div>
                  <div><dt>연동</dt><dd>API 모듈을 기능별로 분리하고 Axios 인터셉터에서 JWT 주입과 만료 토큰 갱신·원 요청 재시도를 처리했습니다. Spring API와 변환 서버의 응답은 업로드→대기→결과 화면으로 구분해 표현했습니다.</dd></div>
                </dl>
                <div class="tag-list"><span v-for="item in ['React', 'Redux Toolkit', 'Axios', 'React Router', 'Tailwind CSS', 'VIEW360']" :key="item">{{ item }}</span></div>
                <div class="project-link-row">
                  <a :href="projectLinks.capeasy" target="_blank" rel="noreferrer" class="text-link"><svg viewBox="0 0 24 24" aria-hidden="true" focusable="false"><path fill-rule="evenodd" d="M12 2C6.48 2 2 6.58 2 12.23c0 4.52 2.87 8.35 6.84 9.71.5.1.68-.22.68-.49 0-.24-.01-1.04-.01-1.89-2.78.62-3.37-1.21-3.37-1.21-.45-1.18-1.11-1.49-1.11-1.49-.91-.64.07-.62.07-.62 1 .07 1.53 1.05 1.53 1.05.89 1.57 2.34 1.11 2.91.85.09-.66.35-1.11.63-1.36-2.22-.26-4.56-1.14-4.56-5.06 0-1.12.39-2.03 1.03-2.75-.1-.26-.45-1.3.1-2.71 0 0 .84-.28 2.75 1.05A9.3 9.3 0 0 1 12 6.97a9.3 9.3 0 0 1 2.5.35c1.91-1.33 2.75-1.05 2.75-1.05.55 1.41.2 2.45.1 2.71.64.72 1.03 1.63 1.03 2.75 0 3.93-2.34 4.79-4.57 5.05.36.32.68.94.68 1.9 0 1.37-.01 2.47-.01 2.81 0 .27.18.59.69.49A10.25 10.25 0 0 0 22 12.23C22 6.58 17.52 2 12 2Z" clip-rule="evenodd"/></svg>GitHub에서 자세히 보기 ↗</a>
                  <a :href="projectLinks.capeasyVideo" target="_blank" rel="noreferrer" class="text-link"><svg viewBox="0 0 24 24" aria-hidden="true" focusable="false"><path d="M23.5 6.2a3 3 0 0 0-2.1-2.1C19.5 3.6 12 3.6 12 3.6s-7.5 0-9.4.5A3 3 0 0 0 .5 6.2 31 31 0 0 0 0 12a31 31 0 0 0 .5 5.8 3 3 0 0 0 2.1 2.1c1.9.5 9.4.5 9.4.5s7.5 0 9.4-.5a3 3 0 0 0 2.1-2.1A31 31 0 0 0 24 12a31 31 0 0 0-.5-5.8ZM9.6 15.6V8.4l6.3 3.6-6.3 3.6Z"/></svg>시연 영상 보기 ↗</a>
                </div>
              </div>
              <div class="case-media-stack">
                <div class="inline-video">
                  <iframe src="https://www.youtube-nocookie.com/embed/1_jV6IfomLs" title="Capeasy 시연 영상" loading="lazy" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
                  <div class="media-caption"><span>PROJECT DEMO</span><p>Capeasy의 제작·공유 흐름을 영상으로 확인할 수 있습니다.</p></div>
                </div>
                <div class="gallery project-gallery" :class="{ 'show-nav': hintVisible.capeasy }" data-gallery="capeasy" role="region" aria-roledescription="carousel" aria-label="Capeasy 주요 화면" tabindex="0" @keydown.left.prevent="moveSlide('capeasy', -1)" @keydown.right.prevent="moveSlide('capeasy', 1)" @touchstart.passive="startSwipe" @touchend.passive="endSwipe('capeasy', $event)" @pointerdown="startMouseDrag" @pointerup="endMouseDrag('capeasy', $event)" @pointercancel="cancelMouseDrag">
                  <div class="slide-counter" aria-live="polite">{{ slideIndexes.capeasy + 1 }} / {{ capeasySlides.length }}</div>
                  <div class="carousel-viewport">
                    <div class="carousel-track" :style="{ transform: `translateX(-${slideIndexes.capeasy * 100}%)` }">
                      <figure v-for="(slide, index) in capeasySlides" :key="slide.src" :aria-hidden="index !== slideIndexes.capeasy">
                        <button class="image-open-button" type="button" :aria-label="`${slide.label} 크게 보기`" :tabindex="index === slideIndexes.capeasy ? 0 : -1" @click="openImage('capeasy', index)">
                          <img :src="slide.src" :alt="slide.alt" loading="lazy" draggable="false">
                        </button>
                        <figcaption>{{ slide.label }}</figcaption>
                      </figure>
                    </div>
                  </div>
                  <button type="button" class="slide-button slide-prev" aria-label="Capeasy 이전 이미지" @click="moveSlide('capeasy', -1)"></button>
                  <button type="button" class="slide-button slide-next" aria-label="Capeasy 다음 이미지" @click="moveSlide('capeasy', 1)"></button>
                  <p v-if="hintVisible.capeasy" class="swipe-hint" aria-hidden="true">↔ 좌우로 밀어보기</p>
                </div>
              </div>
            </article>

            <article class="case-study detailed-case">
              <div class="case-copy">
                <div class="case-meta"><span>03</span><p>2023.09 · Frontend</p></div>
                <h3>Project WorldCloud</h3>
                <p class="case-kicker">뉴스 기반 워드클라우드 서비스</p>
                <p class="case-lead">원하는 날짜와 뉴스 분야를 선택하면 당일 기사를 수집하고, LDA 분석 결과를 워드클라우드와 pyLDAvis로 보여주는 서비스입니다.</p>
                <dl class="case-details">
                  <div><dt>문제</dt><dd>하루 동안 쏟아지는 기사에서 핵심 이슈를 빠르게 파악하고, 관심 키워드의 근거가 된 원문까지 자연스럽게 탐색할 방법이 필요했습니다.</dd></div>
                  <div><dt>역할</dt><dd>HTML·CSS·JavaScript 기반 UI 전반을 담당하고, Spring 백엔드와 Python 뉴스 크롤링·분석 흐름의 연동을 보조했습니다.</dd></div>
                  <div><dt>입력</dt><dd>8개 대주제와 세부 주제, 날짜를 단계적으로 선택하도록 구성하고 입력 검증과 분석 대기 애니메이션으로 처리 상태를 안내했습니다.</dd></div>
                  <div><dt>분석</dt><dd>선택 조건을 Spring Controller에 전달하고, Python이 기사 수집·형태소 추출·Coherence Score 비교 후 생성한 워드클라우드와 pyLDAvis 결과를 화면에 연결했습니다.</dd></div>
                  <div><dt>결과</dt><dd>분석에 사용된 뉴스 원문과 네이버 추가 검색으로 탐색을 확장하고, 데이터 없음이나 분석 실패 시 별도 오류 화면으로 이탈 상황을 처리했습니다.</dd></div>
                </dl>
                <div class="tag-list"><span v-for="item in ['HTML5', 'CSS3', 'JavaScript', 'Spring Boot', 'Thymeleaf', 'Python · LDA']" :key="item">{{ item }}</span></div>
                <div class="project-link-row">
                  <a :href="projectLinks.worldCloud" target="_blank" rel="noreferrer" class="text-link"><svg viewBox="0 0 24 24" aria-hidden="true" focusable="false"><path fill-rule="evenodd" d="M12 2C6.48 2 2 6.58 2 12.23c0 4.52 2.87 8.35 6.84 9.71.5.1.68-.22.68-.49 0-.24-.01-1.04-.01-1.89-2.78.62-3.37-1.21-3.37-1.21-.45-1.18-1.11-1.49-1.11-1.49-.91-.64.07-.62.07-.62 1 .07 1.53 1.05 1.53 1.05.89 1.57 2.34 1.11 2.91.85.09-.66.35-1.11.63-1.36-2.22-.26-4.56-1.14-4.56-5.06 0-1.12.39-2.03 1.03-2.75-.1-.26-.45-1.3.1-2.71 0 0 .84-.28 2.75 1.05A9.3 9.3 0 0 1 12 6.97a9.3 9.3 0 0 1 2.5.35c1.91-1.33 2.75-1.05 2.75-1.05.55 1.41.2 2.45.1 2.71.64.72 1.03 1.63 1.03 2.75 0 3.93-2.34 4.79-4.57 5.05.36.32.68.94.68 1.9 0 1.37-.01 2.47-.01 2.81 0 .27.18.59.69.49A10.25 10.25 0 0 0 22 12.23C22 6.58 17.52 2 12 2Z" clip-rule="evenodd"/></svg>GitHub에서 자세히 보기 ↗</a>
                </div>
              </div>
              <div class="case-media-stack single-media">
                <div class="gallery project-gallery" :class="{ 'show-nav': hintVisible.worldCloud }" data-gallery="worldCloud" role="region" aria-roledescription="carousel" aria-label="Project WorldCloud 주요 화면" tabindex="0" @keydown.left.prevent="moveSlide('worldCloud', -1)" @keydown.right.prevent="moveSlide('worldCloud', 1)" @touchstart.passive="startSwipe" @touchend.passive="endSwipe('worldCloud', $event)" @pointerdown="startMouseDrag" @pointerup="endMouseDrag('worldCloud', $event)" @pointercancel="cancelMouseDrag">
                  <div class="slide-counter" aria-live="polite">{{ slideIndexes.worldCloud + 1 }} / {{ worldCloudSlides.length }}</div>
                  <div class="carousel-viewport">
                    <div class="carousel-track" :style="{ transform: `translateX(-${slideIndexes.worldCloud * 100}%)` }">
                      <figure v-for="(slide, index) in worldCloudSlides" :key="slide.src" :aria-hidden="index !== slideIndexes.worldCloud">
                        <button class="image-open-button" type="button" :aria-label="`${slide.label} 크게 보기`" :tabindex="index === slideIndexes.worldCloud ? 0 : -1" @click="openImage('worldCloud', index)">
                          <img :src="slide.src" :alt="slide.alt" loading="lazy" draggable="false">
                        </button>
                        <figcaption>{{ slide.label }}</figcaption>
                      </figure>
                    </div>
                  </div>
                  <button type="button" class="slide-button slide-prev" aria-label="WorldCloud 이전 이미지" @click="moveSlide('worldCloud', -1)"></button>
                  <button type="button" class="slide-button slide-next" aria-label="WorldCloud 다음 이미지" @click="moveSlide('worldCloud', 1)"></button>
                  <p v-if="hintVisible.worldCloud" class="swipe-hint" aria-hidden="true">↔ 좌우로 밀어보기</p>
                </div>
              </div>
            </article>
          </div>
        </div>
      </section>

      <section id="pangyeori" class="now-section">
        <div class="content now-grid">
          <div class="now-visual">
            <p class="eyebrow">Toy Project</p>
            <button class="now-image-button" type="button" aria-label="판겨리 소개 화면 크게 보기" @click="openImage('pangyeori', 0)">
              <img src="/projects/pangyeori/toy-project.png" alt="AI가 판정하는 1대1 토론 서비스 판겨리 소개 화면" loading="lazy">
            </button>
          </div>
          <div>
            <p class="case-period">2026.08.06 — 진행 중 · Frontend</p>
            <h2>판겨리</h2>
            <p class="now-title">AI가 판정하는 1:1 채팅 토론 서비스</p>
            <p class="now-copy">토론 참여 전후의 상태가 복잡한 서비스에서 사용자가 지금 해야 할 행동을 놓치지 않도록 화면 흐름을 설계하고 있습니다.</p>
            <dl class="now-details">
              <div><dt>구현 완료</dt><dd>이메일 인증과 계정 관리, 토론방 생성·진영 선택, 초대 링크, 상대 참여 확인과 실시간 대기 알림, 프로필·내 토론 목록</dd></div>
              <div><dt>개발 중</dt><dd>단계별 실시간 토론 채팅, 발언 순서와 제한 시간, AI 분석을 통한 최종 판정·요약·피드백</dd></div>
            </dl>
            <div class="tag-list"><span v-for="item in ['Next.js', 'React', 'TypeScript', 'TanStack Query', 'React Hook Form', 'Zod']" :key="item">{{ item }}</span></div>
            <a :href="projectLinks.pangyeori" target="_blank" rel="noreferrer" class="button button-primary"><svg viewBox="0 0 24 24" aria-hidden="true" focusable="false"><path fill-rule="evenodd" d="M12 2C6.48 2 2 6.58 2 12.23c0 4.52 2.87 8.35 6.84 9.71.5.1.68-.22.68-.49 0-.24-.01-1.04-.01-1.89-2.78.62-3.37-1.21-3.37-1.21-.45-1.18-1.11-1.49-1.11-1.49-.91-.64.07-.62.07-.62 1 .07 1.53 1.05 1.53 1.05.89 1.57 2.34 1.11 2.91.85.09-.66.35-1.11.63-1.36-2.22-.26-4.56-1.14-4.56-5.06 0-1.12.39-2.03 1.03-2.75-.1-.26-.45-1.3.1-2.71 0 0 .84-.28 2.75 1.05A9.3 9.3 0 0 1 12 6.97a9.3 9.3 0 0 1 2.5.35c1.91-1.33 2.75-1.05 2.75-1.05.55 1.41.2 2.45.1 2.71.64.72 1.03 1.63 1.03 2.75 0 3.93-2.34 4.79-4.57 5.05.36.32.68.94.68 1.9 0 1.37-.01 2.47-.01 2.81 0 .27.18.59.69.49A10.25 10.25 0 0 0 22 12.23C22 6.58 17.52 2 12 2Z" clip-rule="evenodd"/></svg>GitHub에서 진행 상황 보기 ↗</a>
          </div>
        </div>
      </section>

      <section id="about" class="section">
        <div class="content about-grid">
          <header><p class="eyebrow">How I Work</p><h2>기술보다 먼저<br>사용 흐름을 봅니다.</h2><p>화려한 효과보다 정보의 우선순위, 일관된 상호작용, 유지할 수 있는 구조를 중요하게 생각합니다.</p></header>
          <div class="skill-list">
            <article v-for="skill in skills" :key="skill.index" class="skill-card">
              <p class="skill-index">{{ skill.index }}</p>
              <div><h3>{{ skill.title }}</h3><p>{{ skill.description }}</p><div class="keyword-list"><span v-for="keyword in skill.keywords" :key="keyword">{{ keyword }}</span></div></div>
            </article>
          </div>
        </div>
      </section>

      <section id="activities" class="section soft-section">
        <div class="content">
          <header class="section-heading">
            <div><p class="eyebrow">Beyond the Project</p><h2>관련 대외 활동</h2></div>
            <p>프로젝트를 발표하고, 논문으로 정리하고, 외부 무대에서 결과를 검증한 경험입니다.</p>
          </header>

          <div class="activity-layout">
            <div class="activity-summary" aria-live="polite">
              <p class="activity-index">{{ String(slideIndexes.activities + 1).padStart(2, '0') }}</p>
              <p class="activity-label">{{ activitySlides[slideIndexes.activities].label }}</p>
              <h3>{{ activitySlides[slideIndexes.activities].title }}</h3>
              <p>{{ activitySlides[slideIndexes.activities].meta }}</p>
              <div v-if="activitySlides[slideIndexes.activities].projectHref || activitySlides[slideIndexes.activities].resourceHref" class="activity-links">
                <a v-if="activitySlides[slideIndexes.activities].projectHref" :href="activitySlides[slideIndexes.activities].projectHref || undefined">{{ activitySlides[slideIndexes.activities].projectLabel }}</a>
                <a v-if="activitySlides[slideIndexes.activities].resourceHref" :href="activitySlides[slideIndexes.activities].resourceHref || undefined" target="_blank" rel="noreferrer">{{ activitySlides[slideIndexes.activities].resourceLabel }} ↗</a>
              </div>
            </div>
            <div class="gallery activity-gallery" :class="{ 'show-nav': hintVisible.activities }" data-gallery="activities" role="region" aria-roledescription="carousel" aria-label="대외 활동 자료" tabindex="0" @keydown.left.prevent="moveSlide('activities', -1)" @keydown.right.prevent="moveSlide('activities', 1)" @touchstart.passive="startSwipe" @touchend.passive="endSwipe('activities', $event)" @pointerdown="startMouseDrag" @pointerup="endMouseDrag('activities', $event)" @pointercancel="cancelMouseDrag">
              <div class="slide-counter" aria-live="polite">{{ slideIndexes.activities + 1 }} / {{ activitySlides.length }}</div>
              <div class="carousel-viewport">
                <div class="carousel-track" :style="{ transform: `translateX(-${slideIndexes.activities * 100}%)` }">
                  <figure v-for="(slide, index) in activitySlides" :key="slide.src" :aria-hidden="index !== slideIndexes.activities">
                    <button class="image-open-button" type="button" :aria-label="`${slide.label} 크게 보기`" :tabindex="index === slideIndexes.activities ? 0 : -1" @click="openImage('activities', index)">
                      <img :src="slide.src" :alt="slide.alt" loading="lazy" draggable="false">
                    </button>
                    <figcaption>{{ slide.label }}</figcaption>
                  </figure>
                </div>
              </div>
              <button type="button" class="slide-button slide-prev" aria-label="대외 활동 이전 이미지" @click="moveSlide('activities', -1)"></button>
              <button type="button" class="slide-button slide-next" aria-label="대외 활동 다음 이미지" @click="moveSlide('activities', 1)"></button>
              <p v-if="hintVisible.activities" class="swipe-hint" aria-hidden="true">↔ 좌우로 밀어보기</p>
            </div>
          </div>
        </div>
      </section>

      <section id="experience" class="section">
        <div class="content">
          <p class="eyebrow">Experience</p><h2>경력</h2>
          <div class="timeline">
            <article v-for="career in careers" :key="career.period">
              <p class="timeline-period">{{ career.period }}</p>
              <div><div class="timeline-title"><h3>{{ career.company }}</h3><p>{{ career.role }}</p></div><p class="timeline-copy">{{ career.description }}</p></div>
            </article>
          </div>
        </div>
      </section>

      <section id="education" class="section soft-section">
        <div class="content">
          <header class="section-heading education-heading">
            <div><p class="eyebrow">Education</p><h2>교육</h2></div>
            <p>보안과 인프라부터 데이터 시각화와 AI 서비스 개발까지, 프로젝트 중심으로 학습했습니다.</p>
          </header>
          <article class="education-card">
            <div class="education-copy">
              <div class="education-meta"><span>우수 수료</span><p>2024.12.02 — 2025.06.05 · 960시간</p></div>
              <h3>SK쉴더스 루키즈 지능형 애플리케이션 개발자 양성과정 2기</h3>
              <p class="education-institution">인포섹 아카데미</p>
              <ul class="education-topics">
                <li>가상 인프라 구축 및 관제·탐지 프로젝트</li>
                <li>사이버 보안 기반 웹 시스템 개발</li>
                <li>보안 데이터 분석·시각화 및 AI 기반 서비스 개발</li>
              </ul>
              <div class="education-project">
                <p>Final Project · Frontend</p>
                <strong>서울시 관광지 혼잡도 관제 시스템</strong>
                <a href="#sentry" class="text-link">프로젝트 보기</a>
              </div>
              <a href="https://drive.google.com/file/d/1_hNfI9ukI49KGrk6fLZlA58TAwEWQHN5/view" target="_blank" rel="noreferrer" class="text-link">수료증 원본 보기 ↗</a>
            </div>
          </article>
        </div>
      </section>

      <section id="credentials" class="section">
        <div class="content">
          <header class="section-heading credentials-heading">
            <div><p class="eyebrow">Credentials</p><h2>자격 · 수상</h2></div>
            <p>프로젝트 과정에서 얻은 결과만 간단히 기록했습니다.</p>
          </header>
          <article class="certificate-card">
            <div><p class="certificate-date">2026.09</p><span>최종합격</span></div>
            <h3>정보처리기사</h3>
            <p>한국산업인력공단</p>
          </article>
          <div class="award-list" aria-label="수상 내역">
            <article v-for="award in awards" :key="award.title">
              <time>{{ award.year }}</time>
              <h3>{{ award.title }}</h3>
              <p>{{ award.detail }}</p>
            </article>
          </div>
        </div>
      </section>

      <section id="contact" class="contact-section grid-backdrop">
        <div class="content">
          <p class="eyebrow">Contact</p>
          <h2>함께 해결할 문제가 있다면 이야기를 나누고 싶습니다.</h2>
          <div class="button-row centered">
            <a href="mailto:ghehf51@naver.com" class="button button-naver">
              <svg viewBox="0 0 24 24" aria-hidden="true" focusable="false"><path d="M3 3h5.2l7.6 10.9V3H21v18h-5.2L8.2 10.1V21H3V3Z"/></svg>
              ghehf51@naver.com
            </a>
            <a href="https://github.com/iseungho" target="_blank" rel="noreferrer" class="button button-secondary">
              <svg viewBox="0 0 24 24" aria-hidden="true" focusable="false"><path fill-rule="evenodd" d="M12 2C6.48 2 2 6.58 2 12.23c0 4.52 2.87 8.35 6.84 9.71.5.1.68-.22.68-.49 0-.24-.01-1.04-.01-1.89-2.78.62-3.37-1.21-3.37-1.21-.45-1.18-1.11-1.49-1.11-1.49-.91-.64.07-.62.07-.62 1 .07 1.53 1.05 1.53 1.05.89 1.57 2.34 1.11 2.91.85.09-.66.35-1.11.63-1.36-2.22-.26-4.56-1.14-4.56-5.06 0-1.12.39-2.03 1.03-2.75-.1-.26-.45-1.3.1-2.71 0 0 .84-.28 2.75 1.05A9.3 9.3 0 0 1 12 6.97a9.3 9.3 0 0 1 2.5.35c1.91-1.33 2.75-1.05 2.75-1.05.55 1.41.2 2.45.1 2.71.64.72 1.03 1.63 1.03 2.75 0 3.93-2.34 4.79-4.57 5.05.36.32.68.94.68 1.9 0 1.37-.01 2.47-.01 2.81 0 .27.18.59.69.49A10.25 10.25 0 0 0 22 12.23C22 6.58 17.52 2 12 2Z" clip-rule="evenodd"/></svg>
              GitHub ↗
            </a>
            <a href="https://ghehf.tistory.com/" target="_blank" rel="noreferrer" class="button button-secondary">
              <svg viewBox="0 0 24 24" aria-hidden="true" focusable="false"><circle cx="5" cy="5" r="3"/><circle cx="12" cy="5" r="3"/><circle cx="19" cy="5" r="3"/><circle cx="12" cy="12" r="3"/><circle cx="12" cy="19" r="3"/></svg>
              Tistory ↗
            </a>
          </div>
        </div>
      </section>
    </main>

    <dialog ref="imageDialog" class="image-dialog" aria-label="이미지 크게 보기" @click.self="closeImage" @keydown.left.prevent="moveModal(-1)" @keydown.right.prevent="moveModal(1)">
      <div v-if="activeModalSlide" class="modal-viewer" @touchstart.passive="startSwipe" @touchend.passive="endModalSwipe">
        <div class="modal-toolbar">
          <p>{{ modalSlide + 1 }} / {{ modalGallery ? gallerySlides[modalGallery].length : 0 }}</p>
          <button type="button" aria-label="이미지 닫기" @click="closeImage">×</button>
        </div>
        <figure>
          <div class="modal-stage">
            <img :src="activeModalSlide.src" :alt="activeModalSlide.alt">
            <button v-if="modalGallery && gallerySlides[modalGallery].length > 1" type="button" class="slide-button modal-nav modal-prev" aria-label="이전 이미지" @click="moveModal(-1)"></button>
            <button v-if="modalGallery && gallerySlides[modalGallery].length > 1" type="button" class="slide-button modal-nav modal-next" aria-label="다음 이미지" @click="moveModal(1)"></button>
          </div>
          <figcaption>{{ activeModalCaption }}</figcaption>
        </figure>
      </div>
    </dialog>

    <footer><p>© 2026 Seungho Lee. All rights reserved.</p></footer>
  </div>
</template>
