<template>
  <AppLoaderFull v-if="loading" />
  <div class="font-haffer z-30 relative">
    <div class="grid grid-cols-12">
      <div class="col-span-12">
        Profile
        <button
          class="ml-2 opacity-50"
          @click="editMode = true"
          v-show="!editMode"
        >
          Edit
        </button>
        <button
          class="ml-2 opacity-50"
          @click="editMode = false"
          v-show="editMode"
        >
          Cancel
        </button>
      </div>
    </div>

    <div class="grid grid-cols-12 pt-12 pb-4">
      <div class="col-span-12">Contact</div>
    </div>

    <div class="grid grid-cols-12">
      <div class="col-span-12 md:pb-4 md:col-span-3 opacity-50">Name</div>
      <div class="col-span-12 pb-4 md:col-span-3 flex">
        <template v-if="!editMode">
          <p class="border-b border-transparent">
            {{ profile.firstName }} {{ profile.lastName }}
          </p>
        </template>
        <template v-else>
          <input
            class="font-haffer bg-transparent block border-b border-black w-full text-black outline-none placeholder-opb"
            v-model="update.firstName"
          />
          <input
            class="font-haffer bg-transparent block border-b border-black w-full text-black outline-none placeholder-opb ml-5"
            v-model="update.lastName"
          />
        </template>
      </div>
    </div>

    <div class="grid grid-cols-12">
      <div class="col-span-12 md:pb-4 md:col-span-3 opacity-50">
        Email Address
      </div>
      <div class="col-span-12 pb-4 md:col-span-3">
        <template v-if="!editMode">
          <p class="border-b border-transparent">
            {{ profile.emailAddress }}
          </p>
        </template>
        <template v-else>
          <input
            class="font-haffer bg-transparent block border-b border-black w-full text-black outline-none placeholder-opb"
            v-model="update.emailAddress"
          />
        </template>
      </div>
    </div>

    <div class="grid grid-cols-12">
      <div class="col-span-12 md:pb-4 md:col-span-3 opacity-50">Username</div>
      <div class="col-span-12 pb-4 md:col-span-3">
        <template v-if="!editMode">
          <input
            type="text"
            disabled
            class="font-haffer bg-transparent block border-black w-full text-black outline-none placeholder-opb"
            v-model="profile.username"
          />
        </template>
        <template v-else>
          <input
            type="text"
            class="font-haffer bg-transparent block border-b border-black w-full text-black outline-none placeholder-opb"
            v-model="update.username"
          />
        </template>
      </div>
    </div>

    <div class="grid grid-cols-12">
      <div class="col-span-12 md:col-span-3 opacity-50">Password</div>
      <div class="col-span-12 md:col-span-3">
        <template v-if="!editMode">
          <input
            type="password"
            disabled
            class="font-haffer bg-transparent block border-black w-full text-black outline-none placeholder-opb"
            v-model="profile.password"
          />
        </template>
        <template v-else>
          <input
            type="text"
            class="font-haffer bg-transparent block border-b border-black w-full text-black outline-none placeholder-opb"
            v-model="update.password"
          />
        </template>
      </div>
    </div>

    <p class="pt-12 pb-4">Delegation</p>

    <div class="grid grid-cols-12">
      <div class="col-span-12 md:pb-4 md:col-span-3 opacity-50">
        <span
          @mouseover="showYourDelegateMessage = true"
          @mouseout="showYourDelegateMessage = false"
          >Your Delegate *</span
        >
      </div>
      <div class="col-span-12 pb-4 md:col-span-9">
        <div class="flex w-full">
          <span class="opacity-50" v-show="showYourDelegateMessage"
            >Delegate your voting power to another member of Salon.</span
          >
          <AppDelegateSelect
            @delegate="updateDelegate"
            :disabled="!editMode"
            :class="{ invisible: showYourDelegateMessage }"
          />
        </div>
      </div>
    </div>
    <div class="grid grid-cols-12">
      <div class="col-span-12 md:col-span-3 opacity-50">
        <span
          @mouseover="showDelegateForMessage = true"
          @mouseout="showDelegateForMessage = false"
          >Delegating For *</span
        >
      </div>
      <div class="col-span-12 md:col-span-9">
        <span class="opacity-50" v-show="showDelegateForMessage"
          >Salon members who delegated their voting power to you.</span
        >
        <AppDelegateFor :class="{ invisible: showDelegateForMessage }" />
      </div>
    </div>

    <p class="pt-12 pb-4">Financial</p>

    <div class="grid grid-cols-12">
      <div class="col-span-12 md:col-span-3 opacity-50">Current Units</div>
      <div class="col-span-12 md:col-span-3">
        {{ decimal.format(profile.units) }}
      </div>
    </div>
    <p class="pt-12 pb-4">More</p>
    <div class="grid grid-cols-12">
      <div class="col-span-12 md:col-span-3 opacity-50">Biography</div>
      <div class="col-span-12 md:col-span-6">
        <template v-if="!editMode">
          <p class="border-b border-transparent mb-2">
            {{ profile.biography }}
          </p>
        </template>
        <template v-else>
          <textarea
            class="font-haffer bg-transparent block border border-black w-full text-black outline-none placeholder-opb p-2 mb-2 w-full"
            v-model="update.biography"
          />
          <!-- Character counter -->
          <div class="flex justify-end mt-2 text-sm text-gray-600">
            {{ update?.biography?.length || 0 }} / 50,000
          </div>
        </template>
      </div>
    </div>

    <router-link
      :to="'/manage/proposals/' + walletAddress"
      class="pt-16 flex items-center opacity-50"
      v-show="!editMode"
    >
      <svg
        xmlns="http://www.w3.org/2000/svg"
        fill="none"
        viewBox="0 0 24 24"
        stroke-width="1.5"
        stroke="currentColor"
        class="w-4 h-4 mr-2"
      >
        <path
          stroke-linecap="round"
          stroke-linejoin="round"
          d="M13.5 4.5L21 12m0 0l-7.5 7.5M21 12H3"
        />
      </svg>
      View My Proposals
    </router-link>

    <div v-show="editMode" class="pt-12">
      <AppButton @click="updateProfile">Save Changes</AppButton>
      <button class="px-4 py-2 bg-white" @click="editMode = false">
        Cancel
      </button>
    </div>
    <!--    <p class="pt-16 pb-4">Biography</p> -->
  </div>
</template>
<script>
import axios from "axios";
import AppDelegateSelect from "@/components/AppDelegateSelect";
import AppDelegateFor from "@/components/AppDelegateFor";
import AppButton from "@/components/AppButton";
import AppLoaderFull from "@/components/AppLoaderFull";
export default {
  components: { AppDelegateSelect, AppDelegateFor, AppButton, AppLoaderFull },
  props: ["profile"],
  data() {
    return {
      showDelegateForMessage: false,
      showYourDelegateMessage: false,
      loading: false,
      update: {},
      editMode: false,
      decimal: new Intl.NumberFormat("en-US", {
        style: "decimal",
      }),
    };
  },
  computed: {
    walletAddress() {
      return this.$store.state.walletAddress;
    },
    login() {
      return this.$store.state.login;
    },
  },
  methods: {
    async updateProfile() {
      this.loading = true;
      try {
        const res = await axios.post(
          process.env.VUE_APP_URI + "/profile/" + this.profile.id,
          this.update
        );
        //Reset Username and Password
        //
        this.$store.commit("setLogin", {
          username: this.update.username,
          password: this.update.password,
        });

        await new Promise((resolve) => setTimeout(resolve, 2000));
        await this.$store.dispatch("connect", this.login);
        await this.reup();
      } catch (e) {
        console.error(e);
      }
      this.editMode = false;
      this.loading = false;
    },
    updateDelegate(d) {
      this.update.delegate = d;
    },
    async reup() {
      this.update.firstName = this.profile.firstName;
      this.update.lastName = this.profile.lastName;
      this.update.emailAddress = this.profile.emailAddress;
      this.update.biography = this.profile.biography;
      this.update.delegate = this.profile.delegate;
      this.update.username = this.profile.username;
      this.update.password = this.profile.password;
    },
  },
  async mounted() {
    await this.reup();
  },
};
</script>
