<script setup lang="ts">
import { ref, onMounted, nextTick, computed } from 'vue';
import RecipientsBadge from '@/components/RecipientsBadge.vue'; // Import the Badge component

const props = defineProps<{
  recipients: string[];
}>();

const containerRef = ref<HTMLElement | null>(null);
const displayText = ref<string>('');
const remainingCount = ref<number>(0);
const showTooltip = ref(false); // Track tooltip visibility

// Compute the full list of recipients for the tooltip
const fullRecipientList = computed(() => props.recipients.join(', '));

const updateDisplayText = () => {
  if (containerRef.value) {
    const container = containerRef.value;
    const containerWidth = container.clientWidth - parseFloat(getComputedStyle(container).paddingLeft) - parseFloat(getComputedStyle(container).paddingRight);
    
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

    while (index < props.recipients.length) {
      const email = props.recipients[index];
      testElement.textContent = finalText + (index > 0 ? ', ' : '') + email;
      const emailWidth = testElement.clientWidth;

      if (emailWidth + badgeWidth > containerWidth) {
        break;
      } else {
        finalText = testElement.textContent || '';
        index++;
      }
    }

    if (index < props.recipients.length) {
      remainingCount.value = props.recipients.length - index;
      finalText += '...';
    } else {
      remainingCount.value = 0;
    }

    displayText.value = finalText;

    document.body.removeChild(testElement);
    document.body.removeChild(badgeElement);
  }
};

onMounted(() => {
  nextTick(() => {
    updateDisplayText();
  });
});
</script>

<template>
  <div ref="containerRef" class="container">
    <p class="emails">{{ displayText }}</p>
    <RecipientsBadge 
  v-if="remainingCount" 
  :numTruncated="remainingCount" 
  class="badge" 
  @mouseover="() => { showTooltip = true; console.log('Mouse Over'); }" 
  @mouseleave="() => { showTooltip = false; console.log('Mouse Leave'); }"
/>
    <!-- Tooltip showing all recipients -->
    <div v-if="showTooltip" class="tooltip">
      {{ fullRecipientList }}
    </div>
  </div>
</template>

<style scoped>
.container {
  display: flex;
  align-items: center;
  width: 100%;
  overflow: hidden;
  position: relative;
  padding: 0 8px; /* Adjust as needed */
  box-sizing: border-box; /* Include padding in width calculation */
}

.emails {
  margin: 0;
  flex-grow: 1;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.badge {
  margin-left: 8px; /* Adjust as needed */
  flex-shrink: 0; /* Ensure the badge does not shrink and is always visible */
  cursor: pointer; /* Show pointer to indicate it's hoverable */
}

.tooltip {
  position: absolute;
  bottom: 100%; /* Position above the badge */
  left: 50%;
  transform: translateX(-50%);
  background-color: #333;
  color: #fff;
  padding: 5px;
  border-radius: 3px;
  white-space: nowrap;
  font-size: 12px;
  z-index: 10;
  pointer-events: none;
  opacity: 0.9;
}
</style>

