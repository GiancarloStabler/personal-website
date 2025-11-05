<script setup>
import { ref, onMounted } from "vue";

let closeTimeout = null;
let pillFadeTimeout = null;
let openMenuTimeout = null;

const items = [
  { label: "about", submenu: ["home", "resume"] }, 
  { label: "coursework", submenu: ["mathematics", "computer science"] }, 
  { label: "projects", submenu: ["my website", "directed reading 2025", "directed reading 2024", "my notes"] }, 
  { label: "personal", submenu: ["travel", "hobbies"] }, 
  { label: "contact", submenu: ["email", "linkedin"] }, 
];
const navRef = ref(null);
const btnRefs = ref([]);
const pill = ref(null);
const pillVisible = ref(false);
const active = ref(null);
const openMenu = ref(null);

// For submenu pill animation
const submenuPills = ref({}); // { index: { width, height, top, left } }
const submenuRefs = ref({});  // { index: [<li elements>] }

onMounted(() => {
  btnRefs.value = navRef.value.querySelectorAll("button");
});

function movePill(index) {
  const el = btnRefs.value[index];
  const rect = el.getBoundingClientRect();
  const navRect = navRef.value.getBoundingClientRect();

  active.value = index;
  pillVisible.value = true

  if (openMenuTimeout) clearTimeout(openMenuTimeout);

  openMenuTimeout = setTimeout(() => {
    openMenu.value = index;
  }, 150);

  pill.value = {
    width: rect.width,
    height: rect.height,
    left: rect.left - navRect.left - 5, // the 5 accounts for horizontal padding so that the pill is correctly centered
    top: rect.top - navRect.top - 5,
  };

  // Cancel any close timer when hovering back in
  if (closeTimeout) clearTimeout(closeTimeout);
  if (pillFadeTimeout) clearTimeout(pillFadeTimeout);
  cancelCloseMenu();
  pillVisible.value = true;
}

function startCloseMenu() {
  // Cancel any pending close
  if (closeTimeout) clearTimeout(closeTimeout);

  // Start a short delay before closing, to allow for smooth transitions
  closeTimeout = setTimeout(() => {
    openMenu.value = null;
    fadeOutPill(); // fade out main pill
  }, 150); // adjust for desired responsiveness
}

function cancelCloseMenu() {
  if (closeTimeout) {
    clearTimeout(closeTimeout);
    closeTimeout = null;
  }
  if (pillFadeTimeout) {
    clearTimeout(pillFadeTimeout);
    pillFadeTimeout = null;
  }
  // ensure pill is visible when re-entering quickly
  pillVisible.value = true;
}

function fadeOutPill() {
  if (!pill.value) return;

  // Freeze the pill's last position so it doesn't shift
  const frozen = { ...pill.value };
  pill.value = frozen;

  // Start fade-out animation
  pillVisible.value = false;
}

function clearPill() {
  active.value = null;
  pill.value = null;
}

// Move submenu pill
function moveSubPill(parentIndex, subIndex, event) {
  const el = event.currentTarget;
  const parent = el.parentElement;
  const rect = el.getBoundingClientRect();
  const parentRect = parent.getBoundingClientRect();

  submenuPills.value[parentIndex] = {
    width: rect.width,
    height: rect.height,
    left: rect.left - parentRect.left - 5,
    top: rect.top - parentRect.top - 5,
  };
  cancelCloseMenu();
}

function clearSubPill(parentIndex) {
  submenuPills.value[parentIndex] = null;
}
</script>

<template>
  <nav
    ref="navRef"
    class="fixed top-5 left-1/2 -translate-x-1/2 flex bg-black/30 backdrop-blur-md text-white z-20 rounded-full border border-gray-200 px-1 py-1"
    @mouseleave="startCloseMenu"
  >
    <!-- Animated pill -->
    <transition
      enter-active-class="transition-all ease-out duration-300"
      leave-active-class="transition-all ease-in duration-300"
      @after-leave="() => { pill.value = null; active.value = null }"
    >
      <div
        v-if="pill"
        class="absolute bg-white/90 rounded-full transition-all duration-250 ease-out" 
        :class="pillVisible ? 'opacity-100 scale-100' : 'opacity-0 scale-95'"
        :style="{
          width: pill.width + 'px',
          height: pill.height + 'px',
          transform: `translateX(${pill.left}px) translateY(${pill.top}px)`,
        }"
      ></div>
    </transition>

    <!-- Nav items -->
    <ul class="relative flex space-x-1 z-10 mix-blend-difference">
      <li
        v-for="(item, index) in items"
        :key="index"
        class="relative"
        @mouseenter="movePill(index)"
        @mouseleave="startCloseMenu"
      >
        <button
          class="px-6 py-2 rounded-full transition-all duration-250"
          
        >
          {{ item.label }}
        </button>

        <!-- Flyout menu -->
        <transition
          enter-active-class="transition ease-out duration-300"
          enter-from-class="opacity-0 -translate-y-2"
          enter-to-class="opacity-100 translate-y-0"
          leave-active-class="transition ease-in duration-200"
          leave-from-class="opacity-100 translate-y-0"
          leave-to-class="opacity-0 -translate-y-2"
        >
          <ul
            v-if="openMenu === index && item.submenu.length"
            class="absolute top-full left-1/2 -translate-x-1/2 mt-3 bg-black/30 backdrop-blur-md text-white rounded-2xl border border-gray-200 px-1 py-1 space-y-1 z-30"
            @mouseenter="cancelCloseMenu"
            @mouseleave="startCloseMenu; clearSubPill(index)"
          >
            <!-- Submenu pill -->
            <div
              v-if="submenuPills[index]"
              class="absolute bg-white/90 rounded-xl transition-all duration-250 ease-out"
              :style="{
                width: submenuPills[index].width + 'px',
                height: submenuPills[index].height + 'px',
                transform: `translateX(${submenuPills[index].left}px) translateY(${submenuPills[index].top}px)`,
              }"
            ></div>

            <li
              v-for="(sub, subIndex) in item.submenu"
              :key="subIndex"
              class="px-4 py-1 rounded-xl cursor-pointer text-center z-30 mix-blend-difference transition-all duration-200 text-nowrap"
              @mouseenter="moveSubPill(index, subIndex, $event)"
            >
              {{ sub }}
            </li>
          </ul>
        </transition>
      </li>
    </ul>
  </nav>
</template>
