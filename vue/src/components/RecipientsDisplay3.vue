<script setup lang="ts">
import { defineProps, ref, onMounted, nextTick } from 'vue';
import RecipientsBadge from '@/components/RecipientsBadge.vue'; // Import the Badge component

const props = defineProps<{
  recipients: string[];
}>();

const containerRef = ref<HTMLElement | null>(null);
const displayText = ref<string>('');
const remainingCount = ref<number>(0);
const tooltipText = ref<string>('');

// Function to update the displayed text based on container width
const updateDisplayText = () => {
  if (containerRef.value) {
    const containerWidth = containerRef.value.clientWidth;
    const testElement = document.createElement('span');
    testElement.style.visibility = 'hidden';
    testElement.style.position = 'absolute';
    testElement.style.whiteSpace = 'nowrap';
    document.body.appendChild(testElement);

    let index = 0;
    let finalText = '';

    while (index < props.recipients.length) {
      const email = props.recipients[index];
      testElement.textContent = finalText + (index > 0 ? ', ' : '') + email;
      const emailWidth = testElement.clientWidth;

      if (emailWidth > containerWidth) {
        break; // Stop if the width exceeds the container width
      } else {
        finalText = testElement.textContent || '';
        index++;
      }
    }

    if (index < props.recipients.length) {
      remainingCount.value = props.recipients.length - index;
      finalText += '...';
      tooltipText.value = props.recipients.join(', '); // Set the tooltip text to show all recipients
    } else {
      remainingCount.value = 0;
      tooltipText.value = ''; // No tooltip if all emails fit
    }

    displayText.value = finalText;
    document.body.removeChild(testElement);
  }
};

// Watch for changes and re-calculate when mounted
onMounted(() => {
  nextTick(() => {
    updateDisplayText();
  });
});
</script>

<template>
  <div ref="containerRef" class="container">
    <p>{{ displayText }}</p>
    <!-- Badge with tooltip functionality -->
    <span v-if="remainingCount" class="badge" :title="tooltipText">
      <RecipientsBadge :numTruncated="remainingCount" />
    </span>
  </div>
</template>

<style scoped>
.container {
  position: relative;
  width: 100%;
  overflow: hidden;
}

.badge {
  display: inline-block;
  position: relative;
  background-color: var(--color-primary);
  color: #fff;
  border-radius: 12px;
  padding: 2px 8px;
  font-size: 12px;
  white-space: nowrap;
  cursor: pointer; /* Show a pointer cursor to indicate it's interactive */
}

/* Tooltip styles */
.badge::after {
  content: attr(title); /* Content for tooltip from title attribute */
  position: absolute;
  top: 100%; /* Position tooltip below the badge */
  left: 50%; /* Center horizontally */
  transform: translateX(-50%); /* Center horizontally */
  padding: 5px;
  border-radius: 4px;
  background-color: #333;
  color: #fff;
  white-space: nowrap;
  opacity: 0;
  transition: opacity 0.3s;
  pointer-events: none; /* Tooltip does not interfere with mouse events */
}

.badge:hover::after {
  opacity: 1; /* Show tooltip on hover */
}
</style>

