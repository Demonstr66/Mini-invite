<script setup>
import { ref, nextTick, onMounted, onUnmounted } from 'vue'
import { content as c } from './content'
const step = ref(0), opening = ref(false), heading = ref(null)
const tg = window.Telegram?.WebApp
let timer
function haptic() { try { tg?.HapticFeedback?.impactOccurred('light') } catch {} }
async function show(value) {
 step.value = value; haptic()
 if (value === 0) tg?.BackButton?.hide(); else tg?.BackButton?.show()
 await nextTick(); heading.value?.focus()
}
function open() {
 if (opening.value) return
 opening.value = true; haptic()
 timer = setTimeout(() => { opening.value = false; show(1) }, matchMedia('(prefers-reduced-motion: reduce)').matches ? 0 : 650)
}
function back() { show(Math.max(0, step.value - 1)) }
onMounted(() => { try { tg?.ready(); tg?.expand(); tg?.setHeaderColor?.('#202722'); tg?.setBackgroundColor?.('#202722'); tg?.BackButton?.onClick(back) } catch {} })
onUnmounted(() => { clearTimeout(timer); tg?.BackButton?.offClick(back); tg?.BackButton?.hide() })
</script>
<template>
 <main class="scene">
  <header class="masthead"><span class="wordmark">{{ c.brand }}</span><span class="edition">{{ c.edition }}</span></header>
  <div class="stage">
   <aside class="side-note" aria-hidden="true">{{ c.series }}</aside>
   <div class="card-wrap" :class="{ opening }">
    <div class="paper-under" aria-hidden="true"></div>
    <section v-if="step === 0" class="card cover" aria-label="Обложка приглашения">
     <button class="cover-hit" @click="open" :disabled="opening" :aria-label="c.open"></button>
     <div class="card-top"><span>{{ c.coverLabel }}</span><span class="mark" aria-hidden="true">↗</span></div>
     <div class="cover-body"><span class="number">{{ c.number }}</span><h1>{{ c.coverTitle }}</h1><p class="cover-note">{{ c.coverNote }}</p></div>
     <div class="cover-bottom"><span>{{ c.open }}</span><span class="round-arrow" aria-hidden="true">↗</span></div>
    </section>
    <section v-else-if="step === 1" class="card paper inside" aria-label="Приглашение">
     <div class="card-top"><span>{{ c.insideLabel }}</span><button class="icon-button" @click="show(0)" :aria-label="c.close">×</button></div>
     <h1 ref="heading" tabindex="-1">{{ c.title }}</h1><p class="description">{{ c.description }}</p>
     <dl class="details"><div><dt>{{ c.dateLabel }}</dt><dd>{{ c.date }}</dd></div><div><dt>{{ c.timeLabel }}</dt><dd>{{ c.time }}</dd></div><div class="place"><dt>{{ c.placeLabel }}</dt><dd>{{ c.place }}<small>{{ c.placeNote }}</small></dd></div></dl>
     <button class="primary" @click="show(2)">{{ c.accept }}<span aria-hidden="true">↗</span></button><p class="fine-print">{{ c.note }}</p>
    </section>
    <section v-else class="card paper confirmed" aria-label="Подтверждение">
     <div class="card-top"><span>{{ c.confirmedLabel }}</span><span class="seal" aria-hidden="true">✓</span></div>
     <div class="confirmed-body"><div class="orbit" aria-hidden="true"><i></i><i></i></div><h1 ref="heading" tabindex="-1">{{ c.confirmedTitle }}</h1><p class="description">{{ c.confirmedText }}</p><p class="summary">{{ c.date }}<span> / </span>{{ c.time }}</p></div>
     <button class="secondary" @click="show(1)">{{ c.back }}<span aria-hidden="true">↗</span></button><p class="fine-print">{{ c.localNote }}</p>
    </section>
   </div>
   <nav class="pagination" aria-label="Этапы приглашения"><span v-for="(label, index) in c.steps" :key="label" :class="{ active: step === index }" :aria-current="step === index ? 'step' : undefined">0{{ index + 1 }} <i>{{ label }}</i></span></nav>
  </div>
  <footer><span>{{ c.footer }}</span><span>{{ c.farewell }}</span></footer>
 </main>
</template>
