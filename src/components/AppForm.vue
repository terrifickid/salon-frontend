<template>
  <AppLoaderFull v-show="processing" />
  <div v-show="!processing" v-if="ready" ref="fullpage">
    <div class="grid grid-cols-12">
      <div class="col-span-12 lg:col-span-8">
        <div class="mb-6">{{ name }}</div>
        <ul class="mb-12" v-show="selectedIndex > -1">
          <li
            v-for="(field, index) in fields"
            :key="index"
            class="inline-block items-center mr-2 cursor-pointer"
            :class="{ 'opacity-50': selectedIndex < index }"
            @click="selectedIndex = index"
          >
            <span class="flex items-center">
              {{ field.name }}
              <svg
                v-if="fields.length - 1 != index"
                xmlns="http://www.w3.org/2000/svg"
                fill="none"
                viewBox="0 0 24 24"
                stroke-width="1.5"
                stroke="currentColor"
                class="w-4 h-4 ml-2"
              >
                <path
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  d="M4.5 12h15m0 0l-6.75-6.75M19.5 12l-6.75 6.75"
                />
              </svg>
            </span>
          </li>
        </ul>
        <div v-show="selectedIndex == -1">
          <FormIntro :name="name" :description="description" @ready="next" />
        </div>
        <div v-show="selectedIndex == 1000">
          <span class="text-green-500">
            <span v-if="name == 'Propose'"
              >Proposal submitted for review by Salon's Financial
              Committee.</span
            >
            <span v-else
              >{{ name }} proposal submitted for review by Salon's Financial
              Committee.</span
            >
          </span>
          <p class="opacity-50 mt-20">
            <router-link
              to="/manage/proposals"
              class="flex items-center"
              v-if="isMember"
            >
              <svg
                xmlns="http://www.w3.org/2000/svg"
                fill="none"
                viewBox="0 0 24 24"
                stroke-width="1.5"
                stroke="currentColor"
                class="w-6 h-6 mr-2"
              >
                <path
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  d="M13.5 4.5L21 12m0 0l-7.5 7.5M21 12H3"
                />
              </svg>
              View all proposals</router-link
            >
          </p>
        </div>
        <template v-for="(field, index) in fields" :key="index">
          <div v-show="selectedIndex == index">
            <FormField
              :field="field"
              @ready="next"
              @back="back"
              :index="index"
              v-model="form[field.id]"
            />
          </div>
        </template>
      </div>
    </div>
  </div>
</template>
<script>
import _ from "lodash";
import axios from "axios";
import FormField from "@/components/Form/FormField.vue";
import FormIntro from "@/components/Form/FormIntro.vue";
import AppLoaderFull from "@/components/AppLoaderFull.vue";
export default {
  components: { FormField, AppLoaderFull, FormIntro },
  props: ["url"],
  emits: ["success", "ready"],
  data() {
    return {
      name: "",
      description: "",
      ready: false,
      processing: true,
      btnTxt: "Submit",
      fields: [],
      form: {},
      sending: false,
      selectedIndex: -1,
    };
  },
  computed: {
    profile() {
      return this.$store.state.profile;
    },
    isMember() {
      return _.get(this.profile, "units");
    },
  },
  methods: {
    async back() {
      console.log("back!");
      this.selectedIndex--;
    },
    async next() {
      if (this.selectedIndex == this.fields.length - 1) {
        if (await this.validate()) {
          this.submitForm();
        }
      } else {
        this.selectedIndex++;
      }
    },
    async validate() {
      console.log("validate form!");
      console.log("next!", this.form);
      var i = 0;
      var error = false;
      for (let field of this.fields) {
        if (this.form[field.id] == "") error = true;
        if (!([field.id] in this.form)) error = true;
        if (field.id.split(0).includes("optional")) error = false;
        if (error) {
          this.selectedIndex = i;
          break;
        }
        i++;
      }
      console.log("error is", error);
      if (error) return false;
      return true;
    },
    async submitForm() {
      this.processing = true;
      console.log("Ran Submit!");
      try {
        this.form.profile = this.profile;
        const res = await axios.post(this.url, this.form);
        if (res.data.result) {
          console.log("success", res.data);
          this.selectedIndex = 1000;
          this.processing = false;
          //this.$store.dispatch("connect");
        } else {
          console.error(res.data.message);
          var error = JSON.parse(res.data.message.message);
          alert(
            "Sorry, there is a problem with your application: " +
              _.get(error, "details.errors[0].details")
          );
          this.processing = false;
        }
      } catch (error) {
        this.processing = false;
        console.log("error", error);
      }
    },
  },
  async mounted() {
    try {
      const res = await axios.get(this.url);
      this.name = res.data.name;
      this.description = res.data.description;
      this.fields = res.data.fields.filter(function (field) {
        var disabled = [
          "votes",
          "kycApproved",
          "kycNotified",
          "units",
          "onboardProposal",
          "profile",
          "role",
          "delegate",
          "biography",
          "prettyId",
          "processState",
          "verified",
          "walletAddress",
          "softInvestmentCommitment",
          "adminApproved",
          "queued",
          "memberSince",
        ];
        if (disabled.includes(field.id)) return false;
        return true;
      });

      const cached = [
        "typeOfEntity",
        "firstName",
        "walletAddress",
        "emailAddress",
        "address",
        "phoneNumber",
        "occupation",
        "accreditationMethod",
        "softInvestmentCommitment",
      ];
      this.fields.map(function (field) {
        if (cached.includes(field.id)) {
          field.cached = true;
        }
      });

      this.$emit("ready");
      this.ready = true;
      this.processing = false;
    } catch (error) {
      console.log("error", error);
    }
  },
};
</script>
