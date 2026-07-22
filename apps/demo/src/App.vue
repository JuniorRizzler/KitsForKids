<script setup>
import { computed, onMounted, ref, watch } from 'vue'

const roles = [
  { id: 'student', label: 'Student', initials: 'AJ' },
  { id: 'parent', label: 'Parent', initials: 'MJ' },
  { id: 'tutor', label: 'Tutor', initials: 'LM' },
  { id: 'admin', label: 'Admin', initials: 'SK' },
]

const navByRole = {
  student: [
    ['home', '⌂', 'Overview'], ['request', '✦', 'Request a tutor'],
    ['sessions', '◷', 'My sessions'], ['messages', '◌', 'Messages'],
    ['learning', '▤', 'Learning library'], ['progress', '↗', 'My progress'],
  ],
  parent: [
    ['home', '⌂', 'Family overview'], ['sessions', '◷', 'Sessions'],
    ['progress', '↗', 'Progress reports'], ['messages', '◌', 'Messages'],
    ['consent', '✓', 'Consent & safety'], ['profile', '◎', 'Student profile'],
  ],
  tutor: [
    ['home', '⌂', 'Tutor dashboard'], ['requests', '✦', 'Student requests'],
    ['sessions', '◷', 'My sessions'], ['students', '♧', 'Assigned students'],
    ['messages', '◌', 'Messages'], ['training', '◇', 'Training center'],
    ['hours', '↗', 'Volunteer hours'], ['materials', '▤', 'Prep materials'],
  ],
  admin: [
    ['home', '⌂', 'Operations'], ['matching', '✦', 'Match approvals'],
    ['tutors', '♧', 'Tutor verification'], ['sessions', '◷', 'Session oversight'],
    ['safety', '◇', 'Safety center'], ['reports', '↗', 'Program reports'],
    ['content', '▤', 'Learning content'], ['settings', '⚙', 'Platform settings'],
  ],
}

const currentRole = ref(localStorage.getItem('kfk-role') || 'student')
const currentPage = ref('home')
const showRoles = ref(false)
const showAccessibility = ref(false)
const showMobileNav = ref(false)
const joinedSession = ref(false)
const requestSent = ref(false)
const newMessage = ref('')
const messages = ref([
  { from: 'Lena', text: 'Great work today! I added two practice problems for our next session.', time: '3:42 PM' },
  { from: 'You', text: 'Thank you! I will try them after dinner.', time: '3:45 PM', mine: true },
])
const accessibility = ref(JSON.parse(localStorage.getItem('kfk-accessibility') || '{"largeText":false,"lowStim":false,"captions":true}'))

const activeRole = computed(() => roles.find((role) => role.id === currentRole.value))
const navItems = computed(() => navByRole[currentRole.value])

watch(currentRole, (role) => {
  localStorage.setItem('kfk-role', role)
  currentPage.value = 'home'
  showRoles.value = false
})
watch(accessibility, (value) => localStorage.setItem('kfk-accessibility', JSON.stringify(value)), { deep: true })

onMounted(() => document.documentElement.classList.toggle('large-text', accessibility.value.largeText))
watch(() => accessibility.value.largeText, (value) => document.documentElement.classList.toggle('large-text', value))
watch(() => accessibility.value.lowStim, (value) => document.documentElement.classList.toggle('low-stim', value))

function go(page) {
  currentPage.value = page
  showMobileNav.value = false
}

function sendMessage() {
  if (!newMessage.value.trim()) return
  messages.value.push({ from: 'You', text: newMessage.value.trim(), time: 'Now', mine: true })
  newMessage.value = ''
}
</script>

<template>
  <div class="app-shell">
    <header class="topbar">
      <button class="mobile-menu" aria-label="Open navigation" @click="showMobileNav = !showMobileNav">☰</button>
      <button class="brand" @click="go('home')" aria-label="Kits for Kids home">
        <span class="brand-mark"><span>K</span></span>
        <span><strong>Kits for Kids</strong><small>Learning without limits</small></span>
      </button>
      <div class="top-actions">
        <span class="demo-pill"><i></i> Demo mode · no sign-in</span>
        <button class="icon-btn" title="Accessibility options" @click="showAccessibility = true">Aa</button>
        <button class="icon-btn notification" title="Notifications">♢<b>3</b></button>
        <div class="role-switcher">
          <button class="profile-button" data-testid="role-switcher" @click="showRoles = !showRoles">
            <span class="avatar">{{ activeRole.initials }}</span>
            <span><small>Viewing as</small><strong>{{ activeRole.label }}</strong></span>
            <span>⌄</span>
          </button>
          <div v-if="showRoles" class="role-menu">
            <p>Explore every portal</p>
            <button
              v-for="role in roles"
              :key="role.id"
              :data-testid="`role-${role.id}`"
              :class="{ selected: role.id === currentRole }"
              @click="currentRole = role.id"
            >
              <span class="avatar small">{{ role.initials }}</span>
              <span><strong>{{ role.label }}</strong><small>{{ role.id === 'student' ? 'Alex Johnson' : role.id === 'parent' ? 'Morgan Johnson' : role.id === 'tutor' ? 'Lena Morales' : 'Samira Khan' }}</small></span>
              <b v-if="role.id === currentRole">✓</b>
            </button>
          </div>
        </div>
      </div>
    </header>

    <aside class="sidebar" :class="{ open: showMobileNav }">
      <div class="user-card">
        <span class="avatar large">{{ activeRole.initials }}</span>
        <div><strong>{{ currentRole === 'student' ? 'Alex Johnson' : currentRole === 'parent' ? 'Morgan Johnson' : currentRole === 'tutor' ? 'Lena Morales' : 'Samira Khan' }}</strong><small>{{ currentRole === 'student' ? 'Grade 8 · Riverside Middle' : currentRole === 'parent' ? 'Guardian of Alex' : currentRole === 'tutor' ? 'Volunteer tutor · Verified' : 'Program administrator' }}</small></div>
      </div>
      <nav aria-label="Portal navigation">
        <button v-for="item in navItems" :key="item[0]" :class="{ active: currentPage === item[0] }" @click="go(item[0])">
          <span>{{ item[1] }}</span>{{ item[2] }}<b v-if="item[0] === 'messages'">2</b>
        </button>
      </nav>
      <div class="support-card">
        <span>♡</span><strong>Need help?</strong><p>Our safety team is always here for you.</p><button @click="go('safety')">Contact support</button>
      </div>
      <button class="sidebar-access" @click="showAccessibility = true">Aa &nbsp; Accessibility options</button>
    </aside>

    <main>
      <!-- STUDENT HOME -->
      <template v-if="currentRole === 'student' && currentPage === 'home'">
        <section class="welcome hero-green">
          <div><span class="eyebrow">Wednesday, July 22</span><h1>Good afternoon, Alex <span>👋</span></h1><p>You’re making great progress in algebra. Keep that momentum going!</p></div>
          <div class="weekly-goal"><div><strong>3 of 4</strong><span>Weekly goal</span></div><svg viewBox="0 0 42 42"><circle cx="21" cy="21" r="15.9"/><circle class="progress-ring" cx="21" cy="21" r="15.9"/></svg></div>
        </section>
        <section class="next-session card featured-card">
          <div class="date-tile"><b>23</b><span>JUL</span></div>
          <div class="session-info"><span class="eyebrow">UP NEXT · TOMORROW AT 4:00 PM</span><h2>Algebra: Linear equations</h2><p><span class="avatar tiny">LM</span> With Lena Morales · 45 minutes</p></div>
          <div class="session-actions"><span class="status ready">● Ready to join</span><button class="primary" @click="joinedSession = true; go('live')">Join session →</button></div>
        </section>
        <div class="dashboard-grid">
          <section class="card quick-help">
            <span class="section-icon amber">✦</span><div><h2>Need help now?</h2><p>Connect with an available tutor in just a few minutes.</p></div>
            <div class="subject-row"><button @click="go('request')"><span>∑</span>Math</button><button @click="go('request')"><span>⚗</span>Science</button><button @click="go('request')"><span>✎</span>English</button><button @click="go('request')"><span>＋</span>More</button></div>
          </section>
          <section class="card progress-card"><div class="card-heading"><div><span class="section-icon mint">↗</span><h2>Your progress</h2></div><button @click="go('progress')">View report</button></div><div class="skill"><div><span>Algebra foundations</span><b>78%</b></div><div><i style="width:78%"></i></div></div><div class="skill"><div><span>Reading comprehension</span><b>64%</b></div><div><i style="width:64%"></i></div></div><div class="skill"><div><span>Life science</span><b>82%</b></div><div><i style="width:82%"></i></div></div></section>
        </div>
        <section class="card lessons"><div class="card-heading"><div><span class="section-icon blue">▤</span><h2>Continue learning</h2></div><button @click="go('learning')">Browse library</button></div><div class="lesson-list"><article><span class="lesson-art math">∑</span><div><small>MATHEMATICS · 12 MIN</small><h3>Solving two-step equations</h3><div class="mini-progress"><i style="width:65%"></i></div><p>65% complete</p></div><button>Continue →</button></article><article><span class="lesson-art science">⚗</span><div><small>SCIENCE · 8 MIN</small><h3>Cells and their functions</h3><div class="mini-progress"><i style="width:25%"></i></div><p>25% complete</p></div><button>Continue →</button></article><article><span class="lesson-art english">✎</span><div><small>ENGLISH · 15 MIN</small><h3>Finding the main idea</h3><span class="status new">New</span></div><button>Start →</button></article></div></section>
      </template>

      <!-- PARENT HOME -->
      <template v-else-if="currentRole === 'parent' && currentPage === 'home'">
        <section class="page-heading"><div><span class="eyebrow">FAMILY PORTAL</span><h1>Alex is building momentum</h1><p>Here’s a clear view of sessions, learning progress, and tutor feedback.</p></div><button class="secondary" @click="go('profile')">Manage student profile</button></section>
        <div class="stats-grid"><article><span>◷</span><div><small>SESSIONS THIS MONTH</small><strong>6</strong><p>4.5 learning hours</p></div></article><article><span>↗</span><div><small>GOALS ON TRACK</small><strong>3 of 4</strong><p class="positive">↑ One improved</p></div></article><article><span>★</span><div><small>AVERAGE ENGAGEMENT</small><strong>92%</strong><p>Based on tutor notes</p></div></article><article><span>✓</span><div><small>CONSENT STATUS</small><strong>Complete</strong><p class="positive">All policies current</p></div></article></div>
        <div class="dashboard-grid parent-grid"><section class="card"><div class="card-heading"><div><span class="section-icon mint">↗</span><h2>Progress snapshot</h2></div><button @click="go('progress')">Full report</button></div><div class="big-chart"><div class="chart-labels"><span>100</span><span>75</span><span>50</span><span>25</span></div><div class="chart-bars"><i style="height:55%"><b>Sep</b></i><i style="height:62%"><b>Oct</b></i><i style="height:69%"><b>Nov</b></i><i style="height:76%"><b>Dec</b></i><i style="height:84%"><b>Jan</b></i></div></div><p class="insight">✦ Alex’s algebra confidence has increased 29% since September.</p></section><section class="card"><div class="card-heading"><div><span class="section-icon blue">◷</span><h2>Upcoming</h2></div><button @click="go('sessions')">All sessions</button></div><div class="compact-session"><div class="date-tile mini"><b>23</b><span>JUL</span></div><div><strong>Algebra with Lena</strong><p>Tomorrow · 4:00 PM · 45 min</p></div><span class="status ready">Confirmed</span></div><div class="compact-session"><div class="date-tile mini blue"><b>26</b><span>JUL</span></div><div><strong>Life science with Noah</strong><p>Saturday · 11:30 AM · 30 min</p></div><span class="status ready">Confirmed</span></div><button class="wide secondary">Request another session</button></section></div>
        <section class="card tutor-note"><div class="avatar large">LM</div><div><span class="eyebrow">LATEST TUTOR NOTE · JULY 20</span><h2>“Alex showed real persistence today.”</h2><p>We practiced graphing linear equations. Alex began checking each answer independently and corrected two mistakes without prompting. Next time we’ll work on word problems.</p><button @click="go('messages')">Message Lena →</button></div></section>
      </template>

      <!-- TUTOR HOME -->
      <template v-else-if="currentRole === 'tutor' && currentPage === 'home'">
        <section class="welcome tutor-hero"><div><span class="eyebrow">VOLUNTEER TUTOR PORTAL</span><h1>Welcome back, Lena</h1><p>Your time has helped 18 students feel more confident this year.</p></div><div class="impact"><strong>42.5</strong><span>hours volunteered</span></div></section>
        <div class="stats-grid"><article><span>◷</span><div><small>UPCOMING SESSIONS</small><strong>3</strong><p>Next: tomorrow at 4 PM</p></div></article><article><span>♧</span><div><small>ASSIGNED STUDENTS</small><strong>4</strong><p>2 ongoing · 2 one-time</p></div></article><article><span>★</span><div><small>SUPERVISOR RATING</small><strong>4.9</strong><p class="positive">Excellent standing</p></div></article><article><span>↗</span><div><small>THIS MONTH</small><strong>8.5 hrs</strong><p>12 sessions completed</p></div></article></div>
        <section class="card request-banner"><div><span class="pulse-dot"></span><div><span class="eyebrow">STUDENT WAITING NOW</span><h2>Maya needs help with Geometry</h2><p>Grade 9 · Angles and parallel lines · English</p></div></div><div><span class="status ready">Good match</span><button class="primary" @click="go('live')">Accept & join →</button></div></section>
        <div class="dashboard-grid"><section class="card"><div class="card-heading"><div><span class="section-icon blue">◷</span><h2>Today’s schedule</h2></div><button @click="go('sessions')">View calendar</button></div><div class="timeline"><article><time>4:00<small>PM</small></time><i></i><div><strong>Alex Johnson · Algebra</strong><p>Linear equations · Ongoing student</p></div><button class="secondary" @click="go('live')">Prepare</button></article><article><time>6:30<small>PM</small></time><i></i><div><strong>Jamie Chen · English</strong><p>Essay structure · One-time help</p></div><button class="secondary">Prepare</button></article></div></section><section class="card"><div class="card-heading"><div><span class="section-icon amber">◇</span><h2>Training</h2></div><button @click="go('training')">Training center</button></div><div class="training-item"><span>✓</span><div><strong>Platform safety</strong><p>Completed · 100%</p></div></div><div class="training-item"><span>✓</span><div><strong>Algebra certification</strong><p>Completed · 92%</p></div></div><div class="training-item current"><span>3</span><div><strong>Supporting hospital learners</strong><p>Module 3 of 4 · 15 min left</p><div class="mini-progress"><i style="width:72%"></i></div></div></div></section></div>
      </template>

      <!-- ADMIN HOME -->
      <template v-else-if="currentRole === 'admin' && currentPage === 'home'">
        <section class="page-heading"><div><span class="eyebrow">PROGRAM OPERATIONS · LIVE</span><h1>Everything looks healthy</h1><p>Monitor tutoring activity, approve matches, and keep every interaction safe.</p></div><button class="primary" @click="go('reports')">Download weekly report</button></section>
        <div class="stats-grid admin-stats"><article><span>●</span><div><small>LIVE SESSIONS</small><strong>12</strong><p class="positive">All monitored</p></div></article><article><span>✦</span><div><small>MATCHES TO REVIEW</small><strong>7</strong><p>3 high-priority</p></div></article><article><span>♧</span><div><small>TUTORS ONLINE</small><strong>38</strong><p>Across 11 subjects</p></div></article><article><span>◇</span><div><small>SAFETY FLAGS</small><strong>2</strong><p>Both low severity</p></div></article></div>
        <div class="dashboard-grid admin-grid"><section class="card matches"><div class="card-heading"><div><span class="section-icon amber">✦</span><h2>Match approvals</h2></div><button @click="go('matching')">Review all 7</button></div><article><div class="match-people"><span class="avatar">AR</span><div><strong>Amir R. <em>Grade 7</em></strong><p>Math · Tuesdays 5–7 PM · Arabic preferred</p></div><span class="match-arrow">→</span><span class="avatar tutor">NP</span><div><strong>Noah Patel <em>Verified</em></strong><p>Math 6–10 · Arabic · 4.9 rating</p></div></div><div class="match-score"><b>96%</b><small>match</small></div><button class="secondary">Review</button><button class="primary small-btn">Approve</button></article><article><div class="match-people"><span class="avatar">KS</span><div><strong>Kiara S. <em>Grade 10</em></strong><p>Chemistry · Weekends · Visual learner</p></div><span class="match-arrow">→</span><span class="avatar tutor">EL</span><div><strong>Eric Lee <em>Verified</em></strong><p>Chemistry 9–12 · Visual · 4.8 rating</p></div></div><div class="match-score"><b>91%</b><small>match</small></div><button class="secondary">Review</button><button class="primary small-btn">Approve</button></article></section><section class="card activity-card"><div class="card-heading"><div><span class="section-icon mint">↗</span><h2>Today’s activity</h2></div></div><div class="activity-metric"><div><span>Sessions completed</span><b>84</b></div><div class="mini-progress"><i style="width:84%"></i></div></div><div class="activity-metric"><div><span>Requests matched</span><b>92%</b></div><div class="mini-progress"><i style="width:92%"></i></div></div><div class="activity-metric"><div><span>Average wait</span><b>3m 12s</b></div><div class="mini-progress"><i style="width:65%"></i></div></div><div class="coverage"><strong>Coverage by subject</strong><p><i class="green"></i>Math <b>14 online</b></p><p><i class="blue-dot"></i>Science <b>9 online</b></p><p><i class="amber-dot"></i>English <b>8 online</b></p></div></section></div>
        <section class="card oversight"><div class="card-heading"><div><span class="section-icon blue">◷</span><h2>Live session oversight</h2></div><button @click="go('sessions')">Open monitoring center</button></div><div class="session-table"><div class="table-head"><span>STUDENT</span><span>TUTOR</span><span>SUBJECT</span><span>DURATION</span><span>STATUS</span><span></span></div><div><span><i class="online"></i> Maya P.</span><span>Lena M.</span><span>Geometry</span><span>18:42</span><span class="status ready">Healthy</span><button>View</button></div><div><span><i class="online"></i> Devon K.</span><span>Priya S.</span><span>Biology</span><span>32:08</span><span class="status warning">Check-in</span><button>View</button></div><div><span><i class="online"></i> Sofia A.</span><span>Eric L.</span><span>English</span><span>09:15</span><span class="status ready">Healthy</span><button>View</button></div></div></section>
      </template>

      <!-- LIVE SESSION -->
      <template v-else-if="currentPage === 'live'">
        <section class="live-header"><div><span class="live-label">● LIVE SESSION</span><h1>Algebra · Linear equations</h1><p>Alex Johnson with Lena Morales</p></div><div class="timer">◷ 28:14</div><button class="danger" @click="go('sessions')">End session</button></section>
        <div class="classroom"><section class="whiteboard"><div class="board-tools"><button class="active">↖</button><button>✎</button><button>T</button><button>▢</button><button>↶</button><button>↷</button><span></span><button>＋ Upload</button></div><div class="board-canvas"><small>SHARED WHITEBOARD</small><div class="equation">3x + 7 = 22</div><div class="work"><p>3x + 7 <s>− 7</s> = 22 <s>− 7</s></p><p>3x = 15</p><p class="answer">x = 5 ✓</p></div><div class="coach-note">Great! What operation did you use first?<span>Lena</span></div></div></section><aside class="session-panel"><div class="participant"><span class="avatar large">LM</span><div><strong>Lena Morales</strong><small>Tutor · Speaking</small></div><span>🎙</span></div><div class="participant"><span class="avatar large student">AJ</span><div><strong>Alex Johnson</strong><small>Student</small></div><span>🎙</span></div><div class="panel-tabs"><button class="active">Chat</button><button>Notes</button><button>Files</button></div><div class="session-chat"><p><strong>Lena</strong><span>Let’s isolate x. What should we do with the +7?</span><small>4:18 PM</small></p><p class="mine"><strong>You</strong><span>Subtract 7 from both sides?</span><small>4:19 PM</small></p><p><strong>Lena</strong><span>Exactly! Show me on the board ✨</span><small>4:19 PM</small></p></div><form class="chat-box" @submit.prevent="sendMessage"><input v-model="newMessage" aria-label="Session message" placeholder="Type a message…"><button>➤</button></form><div v-if="accessibility.captions" class="captions"><b>CC</b><span><strong>Lena:</strong> Now divide both sides by three.</span></div><button class="report-link">⚑ Report a safety concern</button></aside></div>
      </template>

      <!-- REQUEST -->
      <template v-else-if="currentPage === 'request' || currentPage === 'matching'">
        <section class="page-heading"><div><span class="eyebrow">{{ currentRole === 'admin' ? 'ADMIN APPROVAL' : 'GET SUPPORT' }}</span><h1>{{ currentRole === 'admin' ? 'Review student–tutor matches' : 'What would you like help with?' }}</h1><p>{{ currentRole === 'admin' ? 'Every suggested match stays pending until an administrator approves it.' : 'Tell us what you need and we’ll find the best available tutor.' }}</p></div></section>
        <section v-if="currentRole !== 'admin'" class="card request-form"><div class="form-step"><span>1</span><div><h2>Choose a subject</h2><div class="choice-grid"><button class="selected">∑<strong>Mathematics</strong></button><button>⚗<strong>Science</strong></button><button>✎<strong>English</strong></button><button>◎<strong>Homework help</strong></button></div></div></div><div class="form-step"><span>2</span><div><h2>Tell your tutor a little more</h2><div class="form-grid"><label>Topic<select><option>Algebra — linear equations</option><option>Geometry</option><option>Fractions</option></select></label><label>Help type<select><option>One-time help</option><option>Ongoing weekly support</option></select></label><label class="full">What are you working on?<textarea placeholder="Example: I’m stuck on questions 4–8 in my worksheet."></textarea></label></div></div></div><div class="form-step"><span>3</span><div><h2>Learning preferences</h2><div class="tag-choices"><button class="selected">Visual examples</button><button>Step-by-step</button><button>Practice together</button><button>Low-stimulation session</button><button>Captions needed</button></div></div></div><button class="primary submit-request" @click="requestSent = true">{{ requestSent ? '✓ Request sent — finding your tutor' : 'Find my tutor →' }}</button></section>
        <section v-else class="card matches expanded"><article v-for="(match, i) in [{s:'Amir R.',d:'Grade 7 · Math · Arabic preferred',t:'Noah Patel',td:'Math 6–10 · Arabic · 4.9 rating',score:'96%'},{s:'Kiara S.',d:'Grade 10 · Chemistry · Visual learner',t:'Eric Lee',td:'Chemistry 9–12 · Visual · 4.8 rating',score:'91%'},{s:'Mateo D.',d:'Grade 6 · English · Ongoing',t:'Jamie Brooks',td:'English 5–9 · Spanish · 4.9 rating',score:'89%'}]" :key="match.s"><div class="match-people"><span class="avatar">{{ match.s.split(' ').map(x=>x[0]).join('') }}</span><div><strong>{{ match.s }}</strong><p>{{ match.d }}</p></div><span class="match-arrow">→</span><span class="avatar tutor">{{ match.t.split(' ').map(x=>x[0]).join('') }}</span><div><strong>{{ match.t }} <em>Verified</em></strong><p>{{ match.td }}</p></div></div><div class="match-score"><b>{{ match.score }}</b><small>match</small></div><button class="secondary">Details</button><button class="primary small-btn">Approve</button></article></section>
      </template>

      <!-- MESSAGES -->
      <template v-else-if="currentPage === 'messages'">
        <section class="page-heading"><div><span class="eyebrow">SUPERVISED MESSAGING</span><h1>Messages</h1><p>All conversations stay on the platform and are reviewed for safety.</p></div></section><div class="messages-layout card"><aside class="conversations"><label>⌕ <input placeholder="Search conversations"></label><button class="active"><span class="avatar">LM</span><span><strong>Lena Morales</strong><small>Great work today! I added…</small></span><time>3:42</time></button><button><span class="avatar tutor">NP</span><span><strong>Noah Patel</strong><small>See you Saturday!</small></span><time>Mon</time></button></aside><section class="conversation"><header><span class="avatar">LM</span><div><strong>Lena Morales</strong><small>Verified tutor · Algebra</small></div><span class="status ready">● Monitored</span></header><div class="message-safety">◇ For safety, never share phone numbers, social accounts, addresses, or private meeting links.</div><div class="message-stream"><article v-for="message in messages" :key="message.text" :class="{ mine: message.mine }"><small>{{ message.from }}</small><p>{{ message.text }}</p><time>{{ message.time }}</time></article></div><form class="message-compose" @submit.prevent="sendMessage"><button type="button">＋</button><input v-model="newMessage" placeholder="Write a message…" aria-label="Message"><button class="primary">Send</button></form></section></div>
      </template>

      <!-- GENERIC RICH PAGES -->
      <template v-else>
        <section class="page-heading"><div><span class="eyebrow">KITS FOR KIDS</span><h1>{{ navItems.find(item => item[0] === currentPage)?.[2] || 'Portal workspace' }}</h1><p>Explore the complete workflow with realistic demonstration data.</p></div><button v-if="currentPage === 'sessions'" class="primary" @click="go('live')">Open live classroom</button></section>
        <div v-if="['learning','training','materials','content'].includes(currentPage)" class="library-grid"><article v-for="course in [{icon:'∑',type:'MATHEMATICS',title:'Solving two-step equations',meta:'12 min · 8 practice questions',p:65},{icon:'⚗',type:'SCIENCE',title:'Cells and their functions',meta:'10 min · Video + worksheet',p:25},{icon:'✎',type:'ENGLISH',title:'Finding the main idea',meta:'15 min · Interactive reading',p:0},{icon:'♡',type:'TUTOR TRAINING',title:'Supporting hospital learners',meta:'20 min · Certificate module',p:72},{icon:'◇',type:'SAFETY',title:'Keeping sessions safe',meta:'Required · 5-question quiz',p:100},{icon:'◎',type:'ACCESSIBILITY',title:'Teaching every kind of learner',meta:'18 min · Practical strategies',p:40}]" :key="course.title" class="card course-card"><span class="lesson-art math">{{course.icon}}</span><small>{{course.type}}</small><h2>{{course.title}}</h2><p>{{course.meta}}</p><div class="mini-progress"><i :style="`width:${course.p}%`"></i></div><footer><span>{{course.p ? course.p+'% complete' : 'Not started'}}</span><button>{{course.p ? 'Continue' : 'Start'}} →</button></footer></article></div>
        <section v-else-if="['progress','reports','hours'].includes(currentPage)" class="card report-page"><div class="report-summary"><div><span class="eyebrow">JULY 2026 REPORT</span><h2>{{ currentRole === 'tutor' ? '42.5 volunteer hours' : 'Strong, steady progress' }}</h2><p>{{ currentRole === 'tutor' ? 'Across 58 completed tutoring sessions and 6 certifications.' : 'Alex attended 92% of scheduled sessions and improved in every active goal.' }}</p></div><div class="report-score"><strong>{{currentRole === 'tutor' ? '58' : '82%'}}</strong><span>{{currentRole === 'tutor' ? 'sessions' : 'overall mastery'}}</span></div></div><div class="report-columns"><article><h3>Highlights</h3><p>✓ Consistent participation</p><p>✓ Algebra goal ahead of schedule</p><p>✓ Strong tutor engagement</p></article><article><h3>Next focus</h3><p>Build confidence with word problems</p><p>Practice explaining each step aloud</p><p>Complete two micro-lessons this week</p></article></div><button class="primary">Download PDF report</button></section>
        <section v-else-if="['sessions','requests','students','tutors','safety'].includes(currentPage)" class="card list-page"><div v-for="(row,i) in 4" :key="row" class="list-row"><span class="date-tile mini" :class="{blue:i%2}"><b>{{23+i*2}}</b><span>JUL</span></span><div><span class="eyebrow">{{ i === 0 ? 'UPCOMING' : i === 1 ? 'COMPLETED' : 'SCHEDULED' }}</span><h3>{{['Algebra · Linear equations','Life science · Cell structure','English · Essay planning','Geometry · Angles'][i]}}</h3><p>{{ currentRole === 'tutor' ? ['Alex Johnson','Maya Patel','Jamie Chen','Amir Rahman'][i] : ['Lena Morales','Noah Patel','Jamie Brooks','Eric Lee'][i] }} · {{30+i*5}} minutes</p></div><span class="status" :class="i===0?'ready':'new'">{{i===0?'Confirmed':'View recap'}}</span><button class="secondary" @click="i===0 && go('live')">{{i===0?'Join':'Details'}}</button></div></section>
        <section v-else class="card settings-page"><div v-for="section in ['Profile and account','Notifications','Consent and privacy','Accessibility preferences','Session safety','Data and reports']" :key="section"><span class="section-icon mint">✓</span><div><h3>{{section}}</h3><p>Configured and ready for this demonstration.</p></div><button>Edit →</button></div></section>
      </template>
    </main>

    <div v-if="showAccessibility" class="modal-backdrop" @click.self="showAccessibility = false">
      <section class="accessibility-panel" role="dialog" aria-modal="true" aria-label="Accessibility options"><header><div><span class="section-icon blue">Aa</span><div><h2>Accessibility</h2><p>Make the portal comfortable for you.</p></div></div><button aria-label="Close" @click="showAccessibility = false">×</button></header><label><span><strong>Larger text</strong><small>Increase text throughout the portal</small></span><input type="checkbox" v-model="accessibility.largeText"><i></i></label><label><span><strong>Low-stimulation mode</strong><small>Reduce color, motion, and visual intensity</small></span><input type="checkbox" v-model="accessibility.lowStim"><i></i></label><label><span><strong>Live captions</strong><small>Show captions automatically in sessions</small></span><input type="checkbox" v-model="accessibility.captions"><i></i></label><button class="primary wide" @click="showAccessibility = false">Save preferences</button></section>
    </div>
  </div>
</template>
