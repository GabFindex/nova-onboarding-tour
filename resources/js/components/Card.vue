<template>
    <VOnboardingWrapper ref="wrapper" :steps="steps" />
    <!-- <div></div> -->
</template>

<script setup>
import { ref, onMounted } from "vue";
import { VOnboardingWrapper, useVOnboarding } from "v-onboarding";
import "v-onboarding/dist/style.css";

const steps = ref();
const wrapper = ref(null);
const { start } = useVOnboarding(wrapper);

function getSeenUris() {
  const seenUris = localStorage.getItem('seenOnboardingUris');
  return seenUris ? JSON.parse(seenUris) : [];
}

function markUriAsSeen(uri) {
  const seenUris = getSeenUris();
  if (!seenUris.includes(uri)) {
    seenUris.push(uri);
    localStorage.setItem('seenOnboardingUris', JSON.stringify(seenUris));
  }
}

onMounted(async () => {
    const currentUrl = window.location.pathname;
    const seenUris = getSeenUris();
    
    if (seenUris.includes(currentUrl)) {
      return;
    }

    let matchingFlow = null;

    async function fetchConfig() {
        console.log("This is a test");
        try {
            const response = await Nova.request().get(
                "/nova-vendor/onboarding/steps".replace("\\\/", "/")
            );

            console.log("response");
            console.log(response);

            const flows = await response.data.flows;

            for (const flowKey in flows) {
                if (flows[flowKey].uri === currentUrl) {
                    matchingFlow = flows[flowKey];
                    break;
                }
            }

            if (matchingFlow) {
                steps.value = Object.values(matchingFlow.steps || {}).map((step) => ({
                    attachTo: { element: step.selector },
                    content: {
                        title: step.title,
                        description: step.description,
                    },
                }));
            }

            console.log("matchingflow", matchingFlow);
            console.log("steps", steps);
        } catch (error) {
            console.error("Error fetching config:", error);
        }
    }

    fetchConfig();

    setTimeout(() => {
        start();
        console.log(steps);
        console.log("test!");

        markUriAsSeen(currentUrl);
    }, 1000);
});
</script>





