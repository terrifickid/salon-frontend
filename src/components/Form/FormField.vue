<template>
  <div>
    <template v-if="isSubmit">
      <InputSubmit
        class="app-frame"
        :count="index + 1"
        :title="getTitle"
        :choices="getChoice"
        :required="true"
        :help="getHelpText"
        @update="updateValue"
        @ready="$emit('ready')"
        @back="$emit('back')"
      />
    </template>
    <template v-else-if="getChoice">
      <InputChoice
        class="app-frame"
        :count="index + 1"
        :title="getTitle"
        :choices="getChoice"
        :id="field.id"
        :cached="field.cached"
        :required="true"
        :help="getHelpText"
        @update="updateValue"
        @ready="$emit('ready')"
        @back="$emit('back')"
      />
    </template>
    <template v-else-if="isUpload">
      <InputUpload
        class="app-frame"
        :count="index + 1"
        :title="getTitle"
        :required="true"
        :help="getHelpText"
        @update="updateValue"
        @ready="$emit('ready')"
        @back="$emit('back')"
      />
    </template>
    <template v-else-if="isLongText">
      <InputLongText
        class="app-frame"
        :count="index + 1"
        :title="getTitle"
        :required="true"
        :help="getHelpText"
        @update="updateValue"
        @ready="$emit('ready')"
        @back="$emit('back')"
      />
    </template>
    <template v-else-if="isUnits">
      <InputUnits
        class="app-frame"
        :count="index + 1"
        :title="getTitle"
        :required="true"
        :help="getHelpText"
        @update="updateValue"
        @ready="$emit('ready')"
        @back="$emit('back')"
      />
    </template>
    <template v-else-if="isMember">
      <InputMember
        class="app-frame"
        :count="index + 1"
        :title="getTitle"
        :required="true"
        :help="getHelpText"
        @update="updateValue"
        @ready="$emit('ready')"
        @back="$emit('back')"
      />
    </template>
    <template v-else>
      <InputText
        class="app-frame"
        :count="index + 1"
        :title="getTitle"
        :id="field.id"
        :cached="field.cached"
        :required="true"
        :help="getHelpText"
        @update="updateValue"
        @ready="$emit('ready')"
        @back="$emit('back')"
      />
    </template>
  </div>
</template>
<script>
import InputText from "@/components/Form/InputText.vue";
import InputLongText from "@/components/Form/InputLongText.vue";
import InputChoice from "@/components/Form/InputChoice.vue";
import InputUpload from "@/components/Form/InputUpload.vue";
import InputUnits from "@/components/Form/InputUnits.vue";
import InputMember from "@/components/Form/InputMember.vue";
import InputSubmit from "@/components/Form/InputSubmit.vue";
export default {
  components: {
    InputText,
    InputLongText,
    InputChoice,
    InputUpload,
    InputUnits,
    InputMember,
    InputSubmit,
  },
  props: ["modelValue", "field", "index", "cache"],
  computed: {
    isMember() {
      var check = this.field.id.split("0");
      return check.includes("member");
    },
    isSubmit() {
      var check = this.field.id.split("0");
      return check.includes("submit");
    },
    isUnits() {
      var check = this.field.id.split("0");
      return check.includes("units");
    },
    isLongText() {
      var check = this.field.id.split("0");
      return check.includes("long");
    },
    getChoice() {
      var choice = this.field.validations.filter(function (v) {
        return "in" in v;
      });
      if (choice.length) return choice[0].in;
      return false;
    },
    getTitle() {
      var title = this.field.validations.filter(function (v) {
        return "prohibitRegexp" in v;
      });
      if (title.length) return title[0].prohibitRegexp.pattern;
      return "No Title";
    },
    getHelpText() {
      var helpText = this.field.validations.filter(function (v) {
        return "prohibitRegexp" in v;
      });
      if (helpText.length) return helpText[0].message;
      return false;
    },
    getId() {
      return this.field.id;
    },
    isUpload() {
      var check = this.field.id.split("0");
      return check.includes("upload");
    },
  },
  methods: {
    updateValue(val) {
      this.$emit("update:modelValue", val);
      if (this.field.cached === true) {
        localStorage.setItem(this.field.id, val);
      }
    },
  },
  mounted() {
    if (this.field.cached && localStorage.getItem(this.field.id) !== null) {
      this.$emit("update:modelValue", localStorage.getItem(this.id));
    }
  },
};
</script>
