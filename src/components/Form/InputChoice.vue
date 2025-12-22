<template>
  <div>
    <FormLabel :count="count" :required="required">{{ title }}</FormLabel>

    <div class="font-haffer">
      <button
        class="px-4 py-2 rounded-full bg-gray-200 hover:bg-gray-300 block mb-2"
        v-for="(choice, index) in choices"
        :key="index"
        @click="
          selectedIndex = index;
          $emit('update', choices[selectedIndex]);
          $emit('ready');
        "
      >
        <span>{{ choice }}</span>
      </button>
    </div>
    <FormHelp :help="help" v-if="help" />
    <FormButtonOk @ready="$emit('ready')" @back="$emit('back')" />
  </div>
</template>
<script>
import FormHelp from "@/components/Form/FormHelp.vue";
import FormLabel from "@/components/Form/FormLabel.vue";
import FormButtonOk from "@/components/Form/FormButtonOk.vue";
export default {
  components: {
    FormLabel,
    FormButtonOk,
    FormHelp,
  },
  props: ["count", "title", "required", "choices", "help", "id", "cached"],
  emits: ["ready"],
  data() {
    return {
      selectedIndex: null,
    };
  },
  methods: {
    getSymbol(i) {
      var symbols = ["A", "B", "C", "D", "E", "F"];
      return symbols[i];
    },
  },
  mounted() {
    if (this.cached === true && localStorage.getItem(this.id) !== null) {
      this.selectedIndex = this.choices.findIndex(
        (el) => el === localStorage.getItem(this.id)
      );
    }
  },
};
</script>
