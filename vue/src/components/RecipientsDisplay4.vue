<script setup lang="ts">
import { ref, onMounted, nextTick } from 'vue';
import RecipientsBadge from '@/components/RecipientsBadge.vue'; // Import the Badge component

const props = defineProps<{
  recipients: string[];
}>();

const containerRef = ref<HTMLElement | null>(null);
const displayText = ref<string>('');
const remainingCount = ref<number>(0);

// Function to update the displayed text based on container width
const updateDisplayText = () => {
  if (containerRef.value) {
    const containerWidth = containerRef.value.clientWidth;
    
    // Create a temporary element to measure text width
    const testElement = document.createElement('span');
    testElement.style.visibility = 'hidden';
    testElement.style.position = 'absolute';
    testElement.style.whiteSpace = 'nowrap';
    document.body.appendChild(testElement);

    // Create a temporary badge to measure its width
    const badgeElement = document.createElement('span');
    badgeElement.style.visibility = 'hidden';
    badgeElement.style.position = 'absolute';
    badgeElement.style.whiteSpace = 'nowrap';
    badgeElement.textContent = '...'; // Simulating the badge content
    document.body.appendChild(badgeElement);

    // Get the badge width
    const badgeWidth = badgeElement.clientWidth;

    let index = 0;
    let finalText = '';

    // Iterate through emails and calculate their combined width
    while (index < props.recipients.length) {
      const email = props.recipients[index];
      testElement.textContent = finalText + (index > 0 ? ', ' : '') + email;
      const emailWidth = testElement.clientWidth;

      // Check if adding this email would exceed the container width minus the badge width
      if (emailWidth + badgeWidth > containerWidth) {
        // If it does, break out of the loop
        break;
      } else {
        finalText = testElement.textContent || '';
        index++;
      }
    }

    // Check if there are remaining emails and set the remaining count
    if (index < props.recipients.length) {
      remainingCount.value = props.recipients.length - index;
      finalText += ', ...';
    } else {
      remainingCount.value = 0;
    }

    displayText.value = finalText;

    document.body.removeChild(testElement);
    document.body.removeChild(badgeElement);
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
    <p class="emails">{{ displayText }}</p>
    <RecipientsBadge v-if="remainingCount" :numTruncated="remainingCount" class="badge" />
  </div>
</template>

<style scoped>
.container {
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

.badge {
  margin-left: 8px; /* Adjust as needed */
}
</style>

