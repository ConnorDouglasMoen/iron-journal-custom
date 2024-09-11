<template>
  <div class="q-pa-sm card-bg rounded-borders">
    <div class="row" v-if="showName">
      <q-input
        class="col-grow"
        name="Name"
        dense
        standout="bg-blue-grey text-white"
        :input-style="{ color: '#ECEFF4' }"
        autogrow
        v-model="data.name"
        debounce="750"
        @update:modelValue="updateValue"
      >
        <template v-slot:append>
          <slot name="action" class="col-shrink" />
        </template>
      </q-input>
    </div>

    <div class="row justify-evenly">
      <q-btn label="Injure" dense :size="btnSize" flat @click="hitDieDecrement" />
      <q-btn flat dense :size="btnSize" :icon="hitDieIcon(value)" /> <!-- change @click to roll dice of that size? -->
      <q-btn label="Bolster" dense :size="btnSize" flat @click="hitDieIncrement" />
      <q-btn icon="mdi-dice-6" flat dense :size="btnSize" @click="conclude"> <!-- Change the @click function to roll for a 1 or 2 -->
        <q-tooltip>Make a killing roll</q-tooltip>
      </q-btn>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, PropType, ref, watch, computed } from 'vue';

import { IEnemy } from 'src/components/models';

import { useQuasar } from 'quasar';

import { moveRoll, NewRollData } from 'src/lib/roll';
import { sleep } from 'src/lib/util';
import { useCampaign } from 'src/store/campaign';

export default defineComponent({
  name: 'EnemyTrack',
  props: {
    modelValue: {
      type: Object as PropType<IEnemy>,
      required: true,
    },
    showName: {
      type: Boolean,
      default: true,
    },
    showDifficulty: {
      type: Boolean,
      default: true,
    },
    setHitDie: {
      type: Number,
      default: 0,
    },
  },
  emits: ['update:modelValue'],
  setup(props, ctx) {
    const data = ref(props.modelValue);
    watch(
      () => props.modelValue,
      () => (data.value = props.modelValue),
      { deep: true }
    );

    const campaign = useCampaign();
    const updateValue = () => ctx.emit('update:modelValue', data.value);

    // Set difficulty externally
    if (props.setHitDie !== 0) {
      // eslint-disable-next-line vue/no-setup-props-destructure
      data.value.hitDie = props.setHitDie;
    }

    const hitDieIncrement = () => {
      if (data.value.hitDie !== 4) {
        data.value.hitDie++;
      }
      updateValue();
    };
    const hitDieDecrement = () => {
      if (data.value.hitDie !== 0) {
        data.value.hitDie--;
      }
      updateValue();
    };
    const hitDieIcon = (value: number): string => {
      switch (value) {
        case 0:
          return 'dice-d4';
        case 1:
          return 'dice-d6';
        case 2:
          return 'dice-d8';
        case 3:
          return 'dice-d12';
        case 4:
          return 'dice-d20';
        default:
          break;
      }

      return 'dice-d20-outline';
    };

    const $q = useQuasar();
    const btnSize = computed(() => {
      if ($q.screen.lt.sm) {
        return 'sm';
      }
      return 'md';
    });

    return {
      data,
      hitDieDecrement,
      hitDieIncrement,
      hitDieIcon,
      updateValue,
      btnSize,
    };
  },
});
</script>
