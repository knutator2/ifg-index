<template>
  <div class="container state" v-if="state">
    <h1 class="headline">{{ state.name }}</h1>
    <div class="prose">
      <div class="text-center italic">
        <p v-if="state.type && state.type !== false">
          <template v-if="state.draft">
            Bisher noch nicht in Kraft getretener Gesetzesentwurf.

            <router-link
              :to="`/laender/${state.draftParent}`"
              class="link whitespace-nowrap"
            >
              Zum aktuellen Gesetz
            </router-link>
          </template>
          <template v-else>
            {{ law }} seit {{ state.year }}.

            <a
              :href="`https://fragdenstaat.de/zustaendigkeit/${state.fds.slug}/`"
              class="link whitespace-nowrap"
            >
              Anfrage stellen
            </a>
          </template>
        </p>
        <p v-else>Kein Informationsfreiheitsgesetz</p>
      </div>
    </div>

    <table class="ranking-bars">
      <tr
        v-for="(bar, i) in performance"
        :key="bar.category.slug"
        :class="{ first: i === 0 }"
      >
        <td>
          <span :class="{ 'font-bold': i === 0 }">
            <router-link
              :to="`#${bar.category.slug}`"
              v-if="i !== 0"
              title="Mehr Informationen..."
            >
              {{ bar.category.title }}

              <i-mdi-information-outline />
            </router-link>
            <span v-else v-text="bar.category.title" />
          </span>
        </td>
        <td class="w-full">
          <ranking-bar :color="bar.category.color" :progress="bar.percentage" />
        </td>
      </tr>
    </table>

    <div class="prose prose-lg my-8">
      <div v-html="state.description" />

      <p>
        <a
          :href="`https://fragdenstaat.de/blog/tag/${
            state.fds.blog || state.fds.slug
          }`"
        >
          → Aktuelles zu {{ state.name }} im FragDenStaat-Blog
        </a>
      </p>
    </div>

    <state-details v-if="state.criteria" :performance="performance" />
    <state-stats :state="state" class="mt-12" />
  </div>
</template>

<script setup>
import { computed, defineProps } from 'vue';
import { useHead } from '@vueuse/head';

import lawtypes from '~/data/lawtypes.yml';
import states from '@data/states';
import _categories from '@data/categories';

const categories = Object.values(_categories);

const props = defineProps({
  state: {
    type: String,
    required: true
  }
});

function getCategory(categorySlug = 'gesamt') {
  return categories.find(c => c.slug === categorySlug);
}

const state = computed(() => states.find(s => s.slug === props.state));
const law = computed(() => lawtypes[state.value.type]);

const performance = computed(() =>
  state.value.performance.map(p => ({
    ...p,
    category: getCategory(p.categorySlug)
  }))
);

const stateName =
  state.value.name === 'Bund' ? 'Deutschland' : state.value.name;
const title = `Informationsfreiheit in ${stateName} - Das Transparenzranking`;
useHead({ title });
</script>

<style scoped>
.state:deep() h2 {
  @apply text-xl md:text-3xl font-semibold mt-6 mb-1;
}
</style>