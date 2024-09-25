<script setup lang="ts">
import { ref, onMounted, nextTick, watch } from 'vue';
import RecipientsBadge from './RecipientsBadge.vue'; // Adjust the path as necessary

const props = defineProps<{
  recipients: string[];
}>();

const containerRef = ref<HTMLElement | null>(null);
const badgeRef = ref<HTMLElement | null>(null);
const displayText = ref<string>('');
const remainingCount = ref<number>(0);
const tooltipText = ref<string>('');
const tooltipVisible = ref<boolean>(false);
const badgeLeft = ref<number>(0);
const badgeTop = ref<number>(0);

const updateDisplayText = () => {
  if (!containerRef.value) return;

  const container = containerRef.value;
  const containerWidth = container.clientWidth;

  const testElement = document.createElement('span');
  testElement.style.visibility = 'hidden';
  testElement.style.position = 'absolute';
  testElement.style.whiteSpace = 'nowrap';
  document.body.appendChild(testElement);

  const badgeElement = document.createElement('span');
  badgeElement.style.visibility = 'hidden';
  badgeElement.style.position = 'absolute';
  badgeElement.style.whiteSpace = 'nowrap';
  badgeElement.textContent = '+99';
  badgeElement.classList.add('badge');
  document.body.appendChild(badgeElement);

  const badgeWidth = badgeElement.clientWidth;

  let index = 0;
  let finalText = '';
  let isTruncated = false;

  while (index < props.recipients.length) {
    const email = props.recipients[index];
    testElement.textContent = finalText + (index > 0 ? ', ' : '') + email;
    const emailWidth = testElement.clientWidth;

    if (emailWidth + badgeWidth > containerWidth) {
      if (index === 0) {
        finalText = email;
      }
      isTruncated = true;
      break;
    } else {
      finalText = testElement.textContent || '';
      index++;
    }
  }

  document.body.removeChild(testElement);

  if (isTruncated) {
    finalText += ', ...';
    remainingCount.value = props.recipients.length - index;
  } else {
    remainingCount.value = 0;
  }

  displayText.value = finalText;
  tooltipText.value = props.recipients.join(', ');
};

const updateTooltipPosition = () => {
  if (badgeRef.value && containerRef.value) {
    const badgeRect = badgeRef.value.getBoundingClientRect();
    const containerRect = containerRef.value.getBoundingClientRect();

    badgeLeft.value = badgeRect.left - containerRect.left + badgeRect.width / 2;
    badgeTop.value = badgeRect.top - containerRect.top + badgeRect.height;
  }
};

const showTooltip = () => {
  console.log('Mouse entered badge - showing tooltip');
  tooltipVisible.value = true;
  updateTooltipPosition();
};

const hideTooltip = () => {
  console.log('Mouse left badge - hiding tooltip');
  tooltipVisible.value = false;
};

// Update display text on mount and resize
onMounted(() => {
  nextTick(() => {
    updateDisplayText();
  });
});

// Update display text on window resize
window.addEventListener('resize', updateDisplayText);

// Watch for changes to remainingCount to update tooltip position
watch(remainingCount, () => {
  updateTooltipPosition();
});
</script>


<template>
  <div 
    ref="containerRef" 
    class="recipients-container"
  >
    <p class="emails">{{ displayText }}</p>
    <!-- Show the badge only if some recipients are trimmed -->
    <RecipientsBadge 
      v-if="remainingCount > 0" 
      :numTruncated="remainingCount" 
      ref="badgeRef"
      @mouseenter="showTooltip"
      @mouseleave="hideTooltip"
    />
    <div 
      v-if="tooltipVisible" 
      class="tooltip"
      :style="{ top: badgeTop + 'px', left: badgeLeft + 'px' }"
    >
      {{ tooltipText }}
    </div>
  </div>
</template>


<style scoped>
.recipients-container {
  display: flex;
  align-items: center;
  width: 100%;
  overflow: hidden;
  position: relative;
}

.emails {
  margin: 0;
  flex-grow: 1;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

/* Tooltip styles */
.tooltip {
  position: fixed; /* Fixed position to keep tooltip in viewport */
  top: 8px; /* Margin from the top */
  right: 8px; /* Margin from the right */
  padding: 8px 16px; /* Padding top and bottom, left and right */
  background-color: #666; /* Background color */
  color: #f0f0f0; /* Text color */
  border-radius: 24px; /* Border radius */
  display: flex; /* Flex display */
  align-items: center; /* Center content vertically */
  opacity: 0;
  visibility: hidden;
  transition: opacity 0.3s, visibility 0.3s; /* Transition for fade-in effect */
  z-index: 9999; /* Ensure the tooltip is above other elements */
}

.recipients-container:hover .tooltip {
  visibility: visible;
  opacity: 1;
}
</style>

