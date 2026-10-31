<script setup>
import { ref, computed, onMounted, onUnmounted, nextTick } from 'vue'
import { content as c } from './content'
const opened = ref(false), confirmed = ref(false), progress = ref(0), journey = ref(null), heading = ref(null)
const tg = window.Telegram?.WebApp
const motionReduced = () => matchMedia('(prefers-reduced-motion: reduce)').matches
const letterStyle = computed(() => ({ '--pull': progress.value, '--release': Math.max(0, (progress.value - .8) / .2) }))
function haptic() { try { tg?.HapticFeedback?.impactOccurred('light') } catch {} }
function syncScroll() {
 if (!opened.value || !journey.value) return
 const distance = Math.max(1, journey.value.offsetHeight - window.innerHeight)
 progress.value = Math.max(0, Math.min(1, (window.scrollY - journey.value.offsetTop) / distance))
}
function open() { opened.value = true; haptic(); tg?.BackButton?.show() }
function extract() {
 const top = journey.value.offsetTop + journey.value.offsetHeight - window.innerHeight
 window.scrollTo({ top, behavior: motionReduced() ? 'instant' : 'smooth' })
}
async function accept() {
 confirmed.value = true; haptic(); await nextTick(); heading.value?.focus({ preventScroll: true })
}
function back() {
 if (confirmed.value) { confirmed.value = false; return }
 opened.value = false; progress.value = 0; window.scrollTo({top:0,behavior:'instant'}); tg?.BackButton?.hide()
}
onMounted(() => {
 window.addEventListener('scroll', syncScroll, { passive: true }); window.addEventListener('resize', syncScroll)
 try { tg?.ready(); tg?.expand(); tg?.disableVerticalSwipes?.(); tg?.setHeaderColor?.('#10100f'); tg?.setBackgroundColor?.('#10100f'); tg?.BackButton?.onClick(back) } catch {}
})
onUnmounted(() => { window.removeEventListener('scroll', syncScroll); window.removeEventListener('resize', syncScroll); tg?.BackButton?.offClick(back); tg?.BackButton?.hide(); tg?.enableVerticalSwipes?.() })
</script>
<template>
 <main ref="journey" class="journey" :class="{ opened }" :style="letterStyle">
  <div class="viewport">
   <header><span>{{ c.brand }}</span><span>{{ c.edition }}</span></header>
   <div class="composition">
    <p class="eyebrow intro">{{ opened ? c.pullHint : c.prelude }}</p>
    <div class="envelope-scene" :class="{ extracted: progress > .85 }">
     <div class="envelope-back" aria-hidden="true"></div>
     <div class="flap" aria-hidden="true"></div>
     <div class="letter-window"><article class="letter" :inert="!opened || progress < .85" :aria-hidden="!opened || progress < .85">
      <div class="letter-frame">
       <template v-if="!confirmed">
        <p class="eyebrow">{{ c.insideLabel }}</p><div class="ornament" aria-hidden="true">◇</div>
        <h1>{{ c.title }}</h1><p class="letter-description">{{ c.description }}</p>
        <dl><div><span class="detail-icon" aria-hidden="true">♧</span><div><dt>{{ c.eventLabel }}</dt><dd>{{ c.event }}</dd></div></div><div><span class="detail-icon" aria-hidden="true">▦</span><div><dt>{{ c.dateLabel }}</dt><dd>{{ c.date }}</dd></div></div><div><span class="detail-icon" aria-hidden="true">◷</span><div><dt>{{ c.timeLabel }}</dt><dd>{{ c.time }}</dd></div></div><div><span class="detail-icon" aria-hidden="true">⌖</span><div><dt>{{ c.placeLabel }}</dt><dd>{{ c.place }}<small>{{ c.placeNote }}</small></dd></div></div></dl>
        <div class="ornament" aria-hidden="true">◇</div><button class="gold-button" @click="accept">{{ c.accept }} <span aria-hidden="true">›</span></button><p class="footnote">{{ c.note }}</p>
       </template>
       <template v-else>
        <p class="eyebrow">{{ c.confirmedLabel }}</p><div class="ornament" aria-hidden="true">◇</div><div class="confirmed-content"><span class="confirmation-mark" aria-hidden="true">✓</span><h1 ref="heading" tabindex="-1">{{ c.confirmedTitle }}</h1><p class="letter-description">{{ c.confirmedText }}</p><p class="summary">{{ c.date }} · {{ c.time }}</p></div><button class="gold-button" @click="confirmed = false">{{ c.back }}</button><p class="local-note">{{ c.localNote }}</p>
       </template>
      </div>
     </article></div>
     <div class="pocket" aria-hidden="true"><div class="pocket-left"></div><div class="pocket-right"></div><div class="pocket-bottom"></div></div>
     <button class="wax-seal" :disabled="opened" @click="open" :aria-label="c.open"><svg viewBox="0 0 48 60" aria-hidden="true"><path d="M15 52Q33 29 28 8M24 33Q9 31 12 21Q24 22 24 33M27 24Q38 20 38 11Q27 13 27 24M19 43Q7 43 8 34Q19 34 19 43M28 14Q18 11 23 3Q30 7 28 14M23 36Q37 36 40 26Q27 26 23 36"/></svg></button>
    </div>
    <div v-if="!opened" class="cover-copy"><h1>{{ c.coverTitle }}</h1><p class="eyebrow">{{ c.coverNote }}</p><div class="ornament" aria-hidden="true">◇</div><button class="gold-button" @click="open">{{ c.open }} <span aria-hidden="true">›</span></button><p class="footnote">{{ c.footer }}</p></div>
    <div v-else class="pull-control" :class="{ hidden: progress > .85 }"><button @click="extract">{{ c.extract }} <span aria-hidden="true">↑</span></button><span class="scroll-track" aria-hidden="true"><i></i></span></div>
   </div>
   <button v-if="opened" class="close-button" @click="back">{{ confirmed ? c.back : c.close }}</button>
  </div>
 </main>
</template>
