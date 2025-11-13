<template>
  <div class="container-fluid px-5">
    <div class="grid grid-cols-12 gap-5">
      <div class="col-span-12 md:col-span-3">
        <div class="md:fixed">
          <ManageNav />
        </div>
      </div>
      <div class="col-span-12 md:col-span-9">
        <div
          v-for="(member, index) in members"
          :key="index"
          class="pb-5 text-left block w-full"
        >
          <div
            class="opacity-50 grid grid-cols-12 gap-x-5 w-full text-left hover:opacity-100 cursor-pointer"
            :class="{ 'opacity-100': selectedIndex == index }"
            @click="selectedIndex = index"
          >
            <div class="col-span-12 md:col-span-4">
              {{ member.fields.firstName }} {{ member.fields.lastName }}
            </div>
            <div class="col-span-12 md:col-span-4">
              Since {{ dateFormated(member.fields.memberSince) }}
            </div>

            <div class="hidden md:block col-span-12 md:col-span-4">
              {{ member.fields.emailAddress }}
            </div>
          </div>
          <div
            class="grid grid-cols-12 gap-5 w-full pt-10 pb-10"
            v-show="selectedIndex == index"
          >
            <div class="col-span-12 md:col-span-3 opacity-50">Biography</div>
            <div class="col-span-12 md:col-span-9">
              {{ member.fields.biography }}
            </div>
            <div class="col-span-12 opacity-50 hover:opacity-100 pt-4">
              <router-link to="/manage/proposals" class="flex items-center">
                <svg
                  xmlns="http://www.w3.org/2000/svg"
                  fill="none"
                  viewBox="0 0 24 24"
                  stroke-width="1.5"
                  stroke="currentColor"
                  class="w-3 h-3 mr-2"
                >
                  <path
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    d="M13.5 4.5L21 12m0 0l-7.5 7.5M21 12H3"
                  />
                </svg>
                <router-link
                  :to="'/manage/proposals/' + member.fields.walletAddress"
                >
                  View {{ member.fields.firstName }}
                  {{ member.fields.lastName }}'s Proposals</router-link
                >
              </router-link>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import axios from "axios";
import ManageNav from "@/components/ManageNav";
export default {
  components: { ManageNav },
  data() {
    return {
      selectedIndex: null,
    };
  },
  methods: {
    dateFormated(date) {
      var d = new Date(date);
      return d.toLocaleString("default", { month: "long", year: "numeric" });
    },
  },
  computed: {
    members() {
      return this.$store.state.members.sort(
        (a, b) =>
          Date.parse(a.fields.memberSince) - Date.parse(b.fields.memberSince)
      );
    },
  },
  async mounted() {
    console.log("members load!");
    try {
      const res = await axios.get(process.env.VUE_APP_URI + "/members");
      this.$store.state.members = res.data;
    } catch (error) {
      console.log("error", error);
    }
  },
};
</script>
