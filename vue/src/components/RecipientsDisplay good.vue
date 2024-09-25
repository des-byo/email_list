<script setup lang="ts">
import { ref, onMounted, nextTick } from 'vue';
import RecipientsBadge from './RecipientsBadge.vue'; // Adjust the path as necessary

const props = defineProps<{
  recipients: string[];
}>();

const containerRef = ref<HTMLElement | null>(null);
const displayText = ref<string>('');
const remainingCount = ref<number>(0);
const tooltipText = ref<string>('');

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

  // Measure and add recipients until they don't fit
  while (index < props.recipients.length) {
    const email = props.recipients[index];
    testElement.textContent = finalText + (index > 0 ? ', ' : '') + email;
    const emailWidth = testElement.clientWidth;

    if (emailWidth + badgeWidth > containerWidth) {
      if (index === 0) {
        // Special case: first email is allowed to be clipped
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
    // If recipients are trimmed, show ", ..." and calculate the remaining count
    finalText += ', ...';
    remainingCount.value = props.recipients.length - index;
  } else {
    remainingCount.value = 0;
  }

  displayText.value = finalText;
  tooltipText.value = props.recipients.join(', ');
};

onMounted(() => {
  nextTick(() => {
    updateDisplayText();
  });
});

window.addEventListener('resize', updateDisplayText);


</script>

<template>
  <div ref="containerRef" class="recipients-container" :title="tooltipText">
    <p class="emails">{{ displayText }}</p>
    <!-- Show the badge only if some recipients are trimmed -->
    <RecipientsBadge v-if="remainingCount > 0" :numTruncated="remainingCount" />
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
</style>

