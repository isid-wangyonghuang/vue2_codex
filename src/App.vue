<template>
  <div class="app-shell">
    <aside class="sidebar">
      <div class="brand">
        <div class="brand-mark">{{ text.brandMark }}</div>
        <div>
          <p>{{ text.appName }}</p>
          <span>{{ text.appSubtitle }}</span>
        </div>
      </div>

      <nav class="side-nav" :aria-label="text.mainNav">
        <button
          v-for="item in navItems"
          :key="item.id"
          type="button"
          :class="{ active: currentView === item.id }"
          @click="currentView = item.id"
          :title="text.nav[item.id]"
        >
          <component :is="item.icon" :size="19" aria-hidden="true" />
          <span>{{ text.nav[item.id] }}</span>
        </button>
      </nav>

      <div class="sidebar-footer">
        <p>{{ text.currentTerm }}</p>
        <strong>{{ text.termName }}</strong>
      </div>
    </aside>

    <main class="workspace">
      <header class="topbar">
        <div>
          <p class="eyebrow">Academic Desk</p>
          <h1>{{ pageTitle }}</h1>
        </div>
        <div class="topbar-actions">
          <label class="language-select">
            <Languages :size="18" aria-hidden="true" />
            <select v-model="language" :aria-label="text.language">
              <option v-for="option in languageOptions" :key="option.value" :value="option.value">
                {{ option.label }}
              </option>
            </select>
          </label>
          <button type="button" class="icon-button" :title="text.exportData" @click="exportStudents">
            <Download :size="19" aria-hidden="true" />
          </button>
          <button type="button" class="primary-button" @click="openCreateModal">
            <Plus :size="18" aria-hidden="true" />
            <span>{{ text.addStudent }}</span>
          </button>
        </div>
      </header>

      <section v-if="currentView === 'overview'" class="content-stack">
        <div class="stats-grid">
          <article v-for="stat in stats" :key="stat.label" class="stat-card">
            <div :class="['stat-icon', stat.tone]">
              <component :is="stat.icon" :size="21" aria-hidden="true" />
            </div>
            <p>{{ stat.label }}</p>
            <strong>{{ stat.value }}</strong>
            <span>{{ stat.note }}</span>
          </article>
        </div>

        <section class="panel">
          <div class="panel-header">
            <div>
              <h2>{{ text.studentList }}</h2>
              <p>{{ text.totalRecords(filteredStudents.length) }}</p>
            </div>
            <div class="filters">
              <label class="search-field">
                <Search :size="18" aria-hidden="true" />
                <input v-model.trim="query" type="search" :placeholder="text.searchPlaceholder" />
              </label>
              <select v-model="classFilter" :aria-label="text.classFilterLabel">
                <option value="all">{{ text.allClasses }}</option>
                <option v-for="className in classes" :key="className" :value="className">{{ className }}</option>
              </select>
              <select v-model="statusFilter" :aria-label="text.statusFilterLabel">
                <option value="all">{{ text.allStatuses }}</option>
                <option v-for="status in statusOptions" :key="status" :value="status">{{ text.status[status] }}</option>
              </select>
            </div>
          </div>

          <div class="table-wrap">
            <table>
              <thead>
                <tr>
                  <th>{{ text.table.student }}</th>
                  <th>{{ text.table.studentNo }}</th>
                  <th>{{ text.table.className }}</th>
                  <th>{{ text.table.score }}</th>
                  <th>{{ text.table.attendance }}</th>
                  <th>{{ text.table.status }}</th>
                  <th>{{ text.table.phone }}</th>
                  <th class="actions-col">{{ text.table.actions }}</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="student in filteredStudents" :key="student.id">
                  <td>
                    <div class="student-cell">
                      <div class="avatar">{{ student.name.slice(0, 1) }}</div>
                      <div>
                        <strong>{{ student.name }}</strong>
                        <span>{{ text.gender[student.gender] }} · {{ text.age(student.age) }}</span>
                      </div>
                    </div>
                  </td>
                  <td>{{ student.studentNo }}</td>
                  <td>{{ student.className }}</td>
                  <td>
                    <div class="score">
                      <span>{{ student.score }}</span>
                      <div class="meter"><i :style="{ width: student.score + '%' }"></i></div>
                    </div>
                  </td>
                  <td>{{ student.attendance }}%</td>
                  <td><span :class="['status-pill', statusTone(student.status)]">{{ text.status[student.status] }}</span></td>
                  <td>{{ student.phone }}</td>
                  <td>
                    <div class="row-actions">
                      <button type="button" :title="text.edit" @click="openEditModal(student)">
                        <Pencil :size="17" aria-hidden="true" />
                      </button>
                      <button type="button" :title="text.delete" @click="removeStudent(student.id)">
                        <Trash2 :size="17" aria-hidden="true" />
                      </button>
                    </div>
                  </td>
                </tr>
                <tr v-if="filteredStudents.length === 0">
                  <td colspan="8" class="empty-state">{{ text.emptyState }}</td>
                </tr>
              </tbody>
            </table>
          </div>
        </section>
      </section>

      <section v-else-if="currentView === 'grades'" class="content-grid">
        <article class="panel chart-panel">
          <div class="panel-header">
            <div>
              <h2>{{ text.scoreDistribution }}</h2>
              <p>{{ text.scoreDistributionNote }}</p>
            </div>
          </div>
          <div class="bars">
            <div v-for="bucket in scoreBuckets" :key="bucket.label" class="bar-row">
              <span>{{ bucket.label }}</span>
              <div class="bar-track"><i :style="{ width: bucket.percent + '%' }"></i></div>
              <strong>{{ bucket.count }}</strong>
            </div>
          </div>
        </article>

        <article class="panel ranking-panel">
          <div class="panel-header">
            <div>
              <h2>{{ text.topScores }}</h2>
              <p>{{ text.topScoresNote }}</p>
            </div>
          </div>
          <div class="ranking-list">
            <div v-for="(student, index) in topStudents" :key="student.id" class="ranking-item">
              <span>{{ index + 1 }}</span>
              <div>
                <strong>{{ student.name }}</strong>
                <p>{{ student.className }}</p>
              </div>
              <em>{{ student.score }}</em>
            </div>
          </div>
        </article>
      </section>

      <section v-else class="content-grid">
        <article class="panel">
          <div class="panel-header">
            <div>
              <h2>{{ text.classOverview }}</h2>
              <p>{{ text.classOverviewNote }}</p>
            </div>
          </div>
          <div class="class-list">
            <div v-for="item in classSummary" :key="item.className" class="class-item">
              <div>
                <strong>{{ item.className }}</strong>
                <span>{{ text.memberCount(item.count) }}</span>
              </div>
              <p>{{ text.points(item.average) }}</p>
            </div>
          </div>
        </article>

        <article class="panel notice-panel">
          <div class="panel-header">
            <div>
              <h2>{{ text.todoTitle }}</h2>
              <p>{{ text.todoNote }}</p>
            </div>
          </div>
          <div class="todo-list">
            <div v-for="todo in todos" :key="todo.title" class="todo-item">
              <component :is="todo.icon" :size="18" aria-hidden="true" />
              <div>
                <strong>{{ todo.title }}</strong>
                <span>{{ todo.desc }}</span>
              </div>
            </div>
          </div>
        </article>
      </section>
    </main>

    <div v-if="isModalOpen" class="modal-backdrop" @click.self="closeModal">
      <form class="student-modal" @submit.prevent="saveStudent">
        <div class="modal-header">
          <div>
            <p class="eyebrow">{{ editingId ? 'Edit Student' : 'New Student' }}</p>
            <h2>{{ editingId ? text.editStudent : text.addStudent }}</h2>
          </div>
          <button type="button" class="icon-button" :title="text.close" @click="closeModal">
            <X :size="19" aria-hidden="true" />
          </button>
        </div>

        <div class="form-grid">
          <label>
            {{ text.form.name }}
            <input v-model.trim="form.name" required />
          </label>
          <label>
            {{ text.form.studentNo }}
            <input v-model.trim="form.studentNo" required />
          </label>
          <label>
            {{ text.form.gender }}
            <select v-model="form.gender">
              <option value="male">{{ text.gender.male }}</option>
              <option value="female">{{ text.gender.female }}</option>
            </select>
          </label>
          <label>
            {{ text.form.age }}
            <input v-model.number="form.age" type="number" min="6" max="40" required />
          </label>
          <label>
            {{ text.form.className }}
            <input v-model.trim="form.className" required />
          </label>
          <label>
            {{ text.form.phone }}
            <input v-model.trim="form.phone" required />
          </label>
          <label>
            {{ text.form.score }}
            <input v-model.number="form.score" type="number" min="0" max="100" required />
          </label>
          <label>
            {{ text.form.attendance }}
            <input v-model.number="form.attendance" type="number" min="0" max="100" required />
          </label>
          <label class="full">
            {{ text.form.status }}
            <select v-model="form.status">
              <option v-for="status in statusOptions" :key="status" :value="status">{{ text.status[status] }}</option>
            </select>
          </label>
        </div>

        <div class="modal-actions">
          <button type="button" class="ghost-button" @click="closeModal">{{ text.cancel }}</button>
          <button type="submit" class="primary-button">
            <Save :size="18" aria-hidden="true" />
            <span>{{ text.save }}</span>
          </button>
        </div>
      </form>
    </div>
  </div>
</template>

<script setup>
import { computed, reactive, ref, watch } from 'vue'
import {
  BookOpen,
  ClipboardList,
  Download,
  GraduationCap,
  Languages,
  LayoutDashboard,
  Pencil,
  Plus,
  Save,
  Search,
  Trash2,
  TriangleAlert,
  UserCheck,
  Users,
  X
} from '@lucide/vue'

const languageStorageKey = 'student-management-language'
const studentStorageKey = 'student-management-vue-data-i18n'
const statusOptions = ['active', 'suspended', 'pending']

const translations = {
  zh: {
    documentTitle: '学生管理系统',
    brandMark: '学',
    appName: '学生管理系统',
    appSubtitle: '教务后台',
    mainNav: '主导航',
    currentTerm: '本学期',
    termName: '2026 春季',
    language: '语言',
    exportData: '导出数据',
    addStudent: '新增学生',
    editStudent: '编辑学生',
    close: '关闭',
    cancel: '取消',
    save: '保存',
    edit: '编辑',
    delete: '删除',
    allClasses: '全部班级',
    allStatuses: '全部状态',
    classFilterLabel: '按班级筛选',
    statusFilterLabel: '按状态筛选',
    searchPlaceholder: '搜索姓名、学号、电话',
    studentList: '学生名单',
    emptyState: '没有找到符合条件的学生',
    scoreDistribution: '成绩分布',
    scoreDistributionNote: '按当前学生名单统计',
    topScores: '成绩前五',
    topScoresNote: '便于快速查看优秀学生',
    classOverview: '班级概览',
    classOverviewNote: '学生人数与平均成绩',
    todoTitle: '待办提醒',
    todoNote: '来自学生状态与出勤数据',
    nav: {
      overview: '学生管理',
      grades: '成绩分析',
      classes: '班级信息'
    },
    table: {
      student: '学生',
      studentNo: '学号',
      className: '班级',
      score: '成绩',
      attendance: '出勤',
      status: '状态',
      phone: '联系电话',
      actions: '操作'
    },
    form: {
      name: '姓名',
      studentNo: '学号',
      gender: '性别',
      age: '年龄',
      className: '班级',
      phone: '联系电话',
      score: '成绩',
      attendance: '出勤率',
      status: '状态'
    },
    gender: {
      male: '男',
      female: '女'
    },
    status: {
      active: '在读',
      suspended: '休学',
      pending: '待确认'
    },
    stats: {
      totalStudents: '学生总数',
      activeStudents: '在读学生',
      averageScore: '平均成绩',
      needsAttention: '需关注',
      classCount: count => `${count} 个班级`,
      normalStatus: '状态正常',
      fullScore: '满分 100',
      warningNote: '成绩或出勤异常'
    },
    todos: {
      lowAttendance: count => `${count} 名学生出勤偏低`,
      lowAttendanceDesc: '建议联系班主任确认情况',
      pendingStatus: count => `${count} 条状态待确认`,
      pendingStatusDesc: '可在学生列表中编辑状态',
      highScore: count => `${count} 名高分学生`,
      highScoreDesc: '适合纳入优秀学生观察名单'
    },
    totalRecords: count => `共 ${count} 条记录`,
    age: value => `${value} 岁`,
    memberCount: count => `${count} 名学生`,
    points: value => `${value} 分`,
    confirmDelete: name => `确定删除 ${name} 的记录吗？`,
    csvName: 'students-zh.csv'
  },
  ja: {
    documentTitle: '学生管理システム',
    brandMark: '学',
    appName: '学生管理システム',
    appSubtitle: '教務管理',
    mainNav: 'メインナビゲーション',
    currentTerm: '今学期',
    termName: '2026 春学期',
    language: '言語',
    exportData: 'データを書き出す',
    addStudent: '学生を追加',
    editStudent: '学生を編集',
    close: '閉じる',
    cancel: 'キャンセル',
    save: '保存',
    edit: '編集',
    delete: '削除',
    allClasses: 'すべてのクラス',
    allStatuses: 'すべての状態',
    classFilterLabel: 'クラスで絞り込み',
    statusFilterLabel: '状態で絞り込み',
    searchPlaceholder: '氏名、学籍番号、電話番号で検索',
    studentList: '学生一覧',
    emptyState: '条件に一致する学生が見つかりません',
    scoreDistribution: '成績分布',
    scoreDistributionNote: '現在の学生一覧をもとに集計',
    topScores: '成績トップ5',
    topScoresNote: '優秀な学生をすばやく確認',
    classOverview: 'クラス概要',
    classOverviewNote: '学生数と平均点',
    todoTitle: '確認事項',
    todoNote: '学生の状態と出席データから作成',
    nav: {
      overview: '学生管理',
      grades: '成績分析',
      classes: 'クラス情報'
    },
    table: {
      student: '学生',
      studentNo: '学籍番号',
      className: 'クラス',
      score: '成績',
      attendance: '出席率',
      status: '状態',
      phone: '電話番号',
      actions: '操作'
    },
    form: {
      name: '氏名',
      studentNo: '学籍番号',
      gender: '性別',
      age: '年齢',
      className: 'クラス',
      phone: '電話番号',
      score: '成績',
      attendance: '出席率',
      status: '状態'
    },
    gender: {
      male: '男',
      female: '女'
    },
    status: {
      active: '在学中',
      suspended: '休学',
      pending: '確認待ち'
    },
    stats: {
      totalStudents: '学生総数',
      activeStudents: '在学中',
      averageScore: '平均点',
      needsAttention: '要確認',
      classCount: count => `${count} クラス`,
      normalStatus: '状態は正常',
      fullScore: '100点満点',
      warningNote: '成績または出席に注意'
    },
    todos: {
      lowAttendance: count => `${count} 名の出席率が低めです`,
      lowAttendanceDesc: '担任への確認をおすすめします',
      pendingStatus: count => `${count} 件の状態確認が必要です`,
      pendingStatusDesc: '学生一覧から状態を編集できます',
      highScore: count => `${count} 名が90点以上です`,
      highScoreDesc: '優秀学生リストの候補になります'
    },
    totalRecords: count => `全 ${count} 件`,
    age: value => `${value} 歳`,
    memberCount: count => `${count} 名`,
    points: value => `${value} 点`,
    confirmDelete: name => `${name} さんの記録を削除しますか？`,
    csvName: 'students-ja.csv'
  },
  en: {
    documentTitle: 'Student Management System',
    brandMark: 'S',
    appName: 'Student Management',
    appSubtitle: 'Academic Office',
    mainNav: 'Main navigation',
    currentTerm: 'Current term',
    termName: 'Spring 2026',
    language: 'Language',
    exportData: 'Export data',
    addStudent: 'Add Student',
    editStudent: 'Edit Student',
    close: 'Close',
    cancel: 'Cancel',
    save: 'Save',
    edit: 'Edit',
    delete: 'Delete',
    allClasses: 'All classes',
    allStatuses: 'All statuses',
    classFilterLabel: 'Filter by class',
    statusFilterLabel: 'Filter by status',
    searchPlaceholder: 'Search name, ID, or phone',
    studentList: 'Student List',
    emptyState: 'No students match the current filters',
    scoreDistribution: 'Score Distribution',
    scoreDistributionNote: 'Calculated from the current student list',
    topScores: 'Top 5 Scores',
    topScoresNote: 'Quickly review high-performing students',
    classOverview: 'Class Overview',
    classOverviewNote: 'Student count and average score',
    todoTitle: 'Follow-ups',
    todoNote: 'Generated from status and attendance data',
    nav: {
      overview: 'Students',
      grades: 'Grades',
      classes: 'Classes'
    },
    table: {
      student: 'Student',
      studentNo: 'Student ID',
      className: 'Class',
      score: 'Score',
      attendance: 'Attendance',
      status: 'Status',
      phone: 'Phone',
      actions: 'Actions'
    },
    form: {
      name: 'Name',
      studentNo: 'Student ID',
      gender: 'Gender',
      age: 'Age',
      className: 'Class',
      phone: 'Phone',
      score: 'Score',
      attendance: 'Attendance Rate',
      status: 'Status'
    },
    gender: {
      male: 'Male',
      female: 'Female'
    },
    status: {
      active: 'Enrolled',
      suspended: 'On Leave',
      pending: 'Pending'
    },
    stats: {
      totalStudents: 'Total Students',
      activeStudents: 'Enrolled',
      averageScore: 'Average Score',
      needsAttention: 'Needs Review',
      classCount: count => `${count} classes`,
      normalStatus: 'Status normal',
      fullScore: 'Out of 100',
      warningNote: 'Score or attendance issue'
    },
    todos: {
      lowAttendance: count => `${count} students have low attendance`,
      lowAttendanceDesc: 'Check in with the homeroom teacher',
      pendingStatus: count => `${count} records need status review`,
      pendingStatusDesc: 'Edit status from the student list',
      highScore: count => `${count} students scored 90 or higher`,
      highScoreDesc: 'Good candidates for the honor list'
    },
    totalRecords: count => `${count} records`,
    age: value => `${value} years old`,
    memberCount: count => `${count} students`,
    points: value => `${value} pts`,
    confirmDelete: name => `Delete ${name}'s record?`,
    csvName: 'students-en.csv'
  }
}

const languageOptions = [
  { value: 'ja', label: '日本語' },
  { value: 'zh', label: '中文' },
  { value: 'en', label: 'English' }
]

const seedStudents = [
  { id: 1, name: '佐藤 蓮', studentNo: 'S2026001', gender: 'male', age: 16, className: '1年A組', score: 92, attendance: 98, status: 'active', phone: '090-0001-2001' },
  { id: 2, name: '鈴木 葵', studentNo: 'S2026002', gender: 'female', age: 15, className: '1年A組', score: 88, attendance: 95, status: 'active', phone: '090-0001-2002' },
  { id: 3, name: '高橋 陽斗', studentNo: 'S2026003', gender: 'male', age: 16, className: '1年B組', score: 76, attendance: 89, status: 'active', phone: '090-0001-2003' },
  { id: 4, name: '田中 美咲', studentNo: 'S2026004', gender: 'female', age: 15, className: '1年B組', score: 95, attendance: 99, status: 'active', phone: '090-0001-2004' },
  { id: 5, name: '伊藤 悠真', studentNo: 'S2026005', gender: 'male', age: 17, className: '2年A組', score: 64, attendance: 82, status: 'pending', phone: '090-0001-2005' },
  { id: 6, name: '山本 結衣', studentNo: 'S2026006', gender: 'female', age: 16, className: '2年A組', score: 84, attendance: 94, status: 'active', phone: '090-0001-2006' },
  { id: 7, name: '中村 大翔', studentNo: 'S2026007', gender: 'male', age: 17, className: '2年B組', score: 58, attendance: 73, status: 'suspended', phone: '090-0001-2007' },
  { id: 8, name: '小林 凛', studentNo: 'S2026008', gender: 'female', age: 16, className: '2年B組', score: 91, attendance: 97, status: 'active', phone: '090-0001-2008' }
]

const normalizeStudent = student => ({
  ...student,
  gender: normalizeGender(student.gender),
  status: normalizeStatus(student.status)
})

const normalizeGender = gender => {
  if (['female', '女'].includes(gender)) return 'female'
  return 'male'
}

const normalizeStatus = status => {
  if (['suspended', '休学', 'On Leave'].includes(status)) return 'suspended'
  if (['pending', '待确认', '確認待ち', 'Pending'].includes(status)) return 'pending'
  return 'active'
}

const loadLanguage = () => {
  const saved = localStorage.getItem(languageStorageKey)
  return translations[saved] ? saved : 'ja'
}

const loadStudents = () => {
  try {
    const saved = localStorage.getItem(studentStorageKey)
    return saved ? JSON.parse(saved).map(normalizeStudent) : seedStudents
  } catch {
    return seedStudents
  }
}

const language = ref(loadLanguage())
const students = ref(loadStudents())
const currentView = ref('overview')
const query = ref('')
const classFilter = ref('all')
const statusFilter = ref('all')
const isModalOpen = ref(false)
const editingId = ref(null)

const emptyForm = () => ({
  name: '',
  studentNo: '',
  gender: 'male',
  age: 16,
  className: '1年A組',
  phone: '',
  score: 80,
  attendance: 95,
  status: 'active'
})

const form = reactive(emptyForm())
const text = computed(() => translations[language.value])

watch(language, value => {
  localStorage.setItem(languageStorageKey, value)
  document.documentElement.lang = value === 'zh' ? 'zh-CN' : value
  document.title = translations[value].documentTitle
}, { immediate: true })

watch(students, value => {
  localStorage.setItem(studentStorageKey, JSON.stringify(value))
}, { deep: true })

const navItems = [
  { id: 'overview', icon: LayoutDashboard },
  { id: 'grades', icon: BookOpen },
  { id: 'classes', icon: ClipboardList }
]

const pageTitle = computed(() => text.value.nav[currentView.value] || text.value.nav.overview)

const classes = computed(() => [...new Set(students.value.map(student => student.className))].sort())

const filteredStudents = computed(() => {
  const keyword = query.value.toLowerCase()
  return students.value.filter(student => {
    const matchQuery = [student.name, student.studentNo, student.phone, student.className].some(value => value.toLowerCase().includes(keyword))
    const matchClass = classFilter.value === 'all' || student.className === classFilter.value
    const matchStatus = statusFilter.value === 'all' || student.status === statusFilter.value
    return matchQuery && matchClass && matchStatus
  })
})

const averageScore = computed(() => {
  if (!students.value.length) return 0
  return Math.round(students.value.reduce((sum, student) => sum + Number(student.score), 0) / students.value.length)
})

const activeCount = computed(() => students.value.filter(student => student.status === 'active').length)
const warningCount = computed(() => students.value.filter(student => student.score < 70 || student.attendance < 85 || student.status !== 'active').length)

const stats = computed(() => [
  { label: text.value.stats.totalStudents, value: students.value.length, note: text.value.stats.classCount(classes.value.length), icon: Users, tone: 'blue' },
  { label: text.value.stats.activeStudents, value: activeCount.value, note: text.value.stats.normalStatus, icon: UserCheck, tone: 'green' },
  { label: text.value.stats.averageScore, value: averageScore.value, note: text.value.stats.fullScore, icon: GraduationCap, tone: 'amber' },
  { label: text.value.stats.needsAttention, value: warningCount.value, note: text.value.stats.warningNote, icon: TriangleAlert, tone: 'red' }
])

const topStudents = computed(() => [...students.value].sort((a, b) => b.score - a.score).slice(0, 5))

const scoreBuckets = computed(() => {
  const buckets = [
    { label: '90-100', min: 90, max: 100 },
    { label: '80-89', min: 80, max: 89 },
    { label: '70-79', min: 70, max: 79 },
    { label: '60-69', min: 60, max: 69 },
    { label: '0-59', min: 0, max: 59 }
  ]
  const total = students.value.length || 1
  return buckets.map(bucket => {
    const count = students.value.filter(student => student.score >= bucket.min && student.score <= bucket.max).length
    return { ...bucket, count, percent: Math.round((count / total) * 100) }
  })
})

const classSummary = computed(() => classes.value.map(className => {
  const members = students.value.filter(student => student.className === className)
  const average = members.length
    ? Math.round(members.reduce((sum, student) => sum + Number(student.score), 0) / members.length)
    : 0
  return { className, count: members.length, average }
}))

const todos = computed(() => [
  {
    title: text.value.todos.lowAttendance(students.value.filter(student => student.attendance < 85).length),
    desc: text.value.todos.lowAttendanceDesc,
    icon: TriangleAlert
  },
  {
    title: text.value.todos.pendingStatus(students.value.filter(student => student.status === 'pending').length),
    desc: text.value.todos.pendingStatusDesc,
    icon: ClipboardList
  },
  {
    title: text.value.todos.highScore(students.value.filter(student => student.score >= 90).length),
    desc: text.value.todos.highScoreDesc,
    icon: GraduationCap
  }
])

const resetForm = values => {
  Object.assign(form, emptyForm(), values)
}

const openCreateModal = () => {
  editingId.value = null
  resetForm()
  isModalOpen.value = true
}

const openEditModal = student => {
  editingId.value = student.id
  resetForm(student)
  isModalOpen.value = true
}

const closeModal = () => {
  isModalOpen.value = false
}

const saveStudent = () => {
  const payload = {
    ...form,
    age: Number(form.age),
    score: Math.min(100, Math.max(0, Number(form.score))),
    attendance: Math.min(100, Math.max(0, Number(form.attendance)))
  }

  if (editingId.value) {
    students.value = students.value.map(student => student.id === editingId.value ? { ...student, ...payload } : student)
  } else {
    students.value = [{ id: Date.now(), ...payload }, ...students.value]
  }

  closeModal()
}

const removeStudent = id => {
  const target = students.value.find(student => student.id === id)
  if (target && window.confirm(text.value.confirmDelete(target.name))) {
    students.value = students.value.filter(student => student.id !== id)
  }
}

const statusTone = status => {
  if (status === 'active') return 'ok'
  if (status === 'suspended') return 'danger'
  return 'pending'
}

const exportStudents = () => {
  const header = [
    text.value.table.student,
    text.value.table.studentNo,
    text.value.form.gender,
    text.value.form.age,
    text.value.table.className,
    text.value.table.score,
    text.value.table.attendance,
    text.value.table.status,
    text.value.table.phone
  ]
  const rows = students.value.map(student => [
    student.name,
    student.studentNo,
    text.value.gender[student.gender],
    student.age,
    student.className,
    student.score,
    student.attendance,
    text.value.status[student.status],
    student.phone
  ])
  const csv = [header, ...rows].map(row => row.map(cell => `"${String(cell).replaceAll('"', '""')}"`).join(',')).join('\n')
  const blob = new Blob([`\uFEFF${csv}`], { type: 'text/csv;charset=utf-8;' })
  const link = document.createElement('a')
  link.href = URL.createObjectURL(blob)
  link.download = text.value.csvName
  link.click()
  URL.revokeObjectURL(link.href)
}
</script>
