<template>
  <AppLoaderFull v-if="!loaded" />
  <AppFileModal :file="fileModal" @close="fileModal = false" v-if="fileModal" />
  <template v-if="loaded">
    <template v-if="isProposer && !isMember && !passed">
      <p class="mb-8">Onboard</p>
      <h1 class="opacity-50 font-haffer">
        Your application is being voted on by Salon members.
      </h1>
    </template>
    <template v-else>
      <div>
        <div class="grid grid-cols-12 mb-4">
          <div
            class="col-span-12 lg:col-span-3 capitalize"
            :class="{ 'text-green-500': !hasPassed }"
          >
            <span class="mr-8" :class="{ dot: !hasPassed }">{{
              prettyId
            }}</span>
            {{ proposalFormat.contentType }}
          </div>
          <div class="col-span-12 lg:col-span-6 py-5 lg:py-0">
            <AppCountdown
              v-if="hasPassed"
              :start="0"
              :type="proposal.sys.contentType.sys.id"
            />
            <AppCountdown
              v-else
              :start="adminApproved"
              :type="proposal.sys.contentType.sys.id"
            />
          </div>
          <div class="col-span-12 lg:col-span-3">
            {{ proposalFormat.profile.firstName }}
            {{ proposalFormat.profile.lastName }}
          </div>
        </div>

        <CounterVote
          :votes="proposalFormat.votes"
          :members="members"
          v-if="isVotable"
        />

        <div
          class="grid grid-cols-12 flex items-center py-5"
          v-if="canVote && isVotable"
        >
          <div class="col-span-4 lg:col-span-2" v-if="!hasPassed">
            Your Vote
          </div>
          <div class="col-span-4 lg:col-span-6 flex">
            <AppButtonVote
              :id="proposalFormat.id"
              :votes="proposalFormat.votes"
              :choice="true"
              :label="'Yes'"
              class="mr-2"
              @voting="handleVoting"
              @voted="handleVoted"
              :processing="isVoting"
            />
            <AppButtonVote
              :id="proposalFormat.id"
              :votes="proposalFormat.votes"
              :choice="false"
              :label="'No'"
              @voting="handleVoting"
              @voted="handleVoted"
              :processing="isVoting"
            />
          </div>
        </div>

        <ul v-if="proposal.sys.contentType.sys.id != 'exchange'">
          <li
            v-for="(field, index) in proposalFormat.fields"
            :key="index"
            class="mr-5 mb-5"
          >
            <template v-if="disabledFields.includes(field.id)"></template>
            <template
              v-else-if="String(field.id).split('0').includes('upload')"
            >
              <span class="opacity-50">{{ field.label }}</span
              ><br />
              <div class="grid grid-cols-12 gap-4">
                <div
                  class="col-span-12 md:col-span-4 lg:col-span-3"
                  v-for="(item, index) in JSON.parse(field.value)"
                  :key="index"
                >
                  <a
                    v-if="item.type.includes('image')"
                    :href="item.url"
                    target="_blank"
                  >
                    <img class="cursor-pointer" :src="item.url" />
                  </a>
                  <a
                    :href="item.url"
                    v-if="item.type.includes('pdf')"
                    class="block w-full aspect-square bg-gray-100 flex items-end p-4 border cursor-pointer"
                    target="_blank"
                  >
                    {{ item.name }} ({{ formatBytes(item.size) }})
                  </a>
                </div>
              </div>
            </template>
            <template v-else-if="String(field.id).split('0').includes('units')">
              <span class="opacity-50">{{ field.label }}</span
              ><br />

              {{ name }} would like to purchase
              {{ getJSON(field.value).units.toLocaleString() }} units at a price
              of
              {{
                format.format(
                  getJSON(field.value).amount / getJSON(field.value).units
                )
              }}
              per unit for a total of
              {{ format.format(getJSON(field.value).amount) }}.

              <p class="text-green-500">
                For reference, the current trade price of Salon units is

                {{ format.format(suggestedTradingPrice) }}.
              </p>
            </template>
            <template
              v-else-if="String(field.id).split('0').includes('member')"
            >
              <span class="opacity-50">{{ field.label }}</span
              ><br />{{ getJSON(field.value).firstName }}
              {{ getJSON(field.value).lastName }}<br />
              <!--  {{ getJSON(field.value).walletAddress }} -->
            </template>
            <template v-else>
              <span class="opacity-50">{{ field.label }}</span
              ><br />
              <div style="white-space: pre-wrap">{{ field.value }}</div>
            </template>
          </li>
        </ul>

        <ExecuteProposal :proposal="proposal" />

        <router-link to="/manage/proposals">
          <button class="flex items-center mt-12 opacity-50">
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
                d="M6 18L18 6M6 6l12 12"
              />
            </svg>

            Close
          </button>
        </router-link>
      </div>
    </template>
  </template>
</template>
<script>
import _ from "lodash";
import axios from "axios";
import AppLoaderFull from "@/components/AppLoaderFull";
import CounterVote from "@/components/CounterVote";
import AppCountdown from "@/components/AppCountdown";
import AppButtonVote from "@/components/AppButtonVote";
import AppFileModal from "@/components/AppFileModal";
import ExecuteProposal from "@/components/execute/ExecuteProposal";

export default {
  components: {
    AppFileModal,
    CounterVote,
    AppCountdown,
    AppButtonVote,
    AppLoaderFull,
    ExecuteProposal,
  },
  props: ["id"],
  emits: ["ready"],
  data() {
    return {
      isVoting: false,
      fileModal: false,
      proposal: {},
      proposalFormat: {},
      members: [],
      loaded: false,
      format: new Intl.NumberFormat("en-US", {
        style: "currency",
        currency: "USD",
      }),
      disabledFields: [
        "profile",
        "Submit Proposal",
        "subscriptionBooklet0upload",
        "votes",
        "submitProposal0submit",
        "profile",
        "prettyId",
        "processState",
        "verified",
        "adminApproved",
        "queued",
      ],
      treasury: {},
    };
  },
  computed: {
    adminApproved() {
      return _.get(this.proposal, "fields.adminApproved", null);
    },
    prettyId() {
      //return this.item.fields.prettyId;
      var zeroes = new Array(3 + 1).join("0");
      return (
        zeroes + this.proposal.fields.prettyId.toString().replace(/\D/g, "")
      ).slice(-3);
    },
    name() {
      return (
        this.proposalFormat.profile.firstName +
        " " +
        this.proposalFormat.profile.lastName
      );
    },
    bookValue() {
      return (
        _.get(this.treasury, "balance") +
        _.get(this.treasury, "balanceInUsdc") +
        _.get(this.treasury, "collectionValue")
      );
    },
    suggestedTradingPrice() {
      return _.get(this.treasury, "currentTradePrice");
    },
    hasPassed() {
      return typeof _.get(this.proposalFormat, "votes.passed") == "boolean";
    },
    isProposer() {
      return true;
    },
    isMember() {
      return this.profile.units;
    },
    isVotable() {
      if (this.proposalFormat.contentType == "exchange") return false;
      return true;
    },
    canVote() {
      if ("units" in this.profile && this.profile.units > 0) return true;
      return false;
    },
    profile() {
      return this.$store.state.profile;
    },
    uri() {
      return process.env.VUE_APP_URI + "/entry/" + this.id;
    },
    passed() {
      return _.get(this.proposalFormat, "votes.passed") === true;
    },
  },
  methods: {
    async handleVoting(label) {
      console.log("voting!", label);
      this.isVoting = label;
    },
    async handleVoted() {
      console.log("voted!");
      await this.init();
      this.isVoting = false;
    },
    handleVote() {},
    formatBytes(a, b = 2) {
      if (!+a) return "0 Bytes";
      const c = 0 > b ? 0 : b,
        d = Math.floor(Math.log(a) / Math.log(1024));
      return `${parseFloat((a / Math.pow(1024, d)).toFixed(c))} ${
        ["Bytes", "KB", "MB", "GB", "TB", "PB", "EB", "ZB", "YB"][d]
      }`;
    },
    getJSON(v) {
      return JSON.parse(v);
    },
    async getMembers() {
      try {
        const res = await axios.get(process.env.VUE_APP_URI + "/members");
        return res.data;
      } catch (error) {
        console.log("error", error);
      }
    },
    getFieldLabel(fields, id) {
      for (let field of fields) {
        if (field.id == id) return field.name;
      }
    },
    async assembleProposalType(type) {
      var scope = this;
      try {
        var e = await axios.get(process.env.VUE_APP_URI + "/form/" + type);

        var votes = { votes: [] };
        if ("votes" in this.proposal.fields) {
          votes = this.proposal.fields.votes;
        }

        var fields = await Object.entries(this.proposal.fields).map(function (
          field
        ) {
          var label = scope.getFieldLabel(e.data.fields, field[0]);
          return {
            label: label,
            id: field[0],
            value: field[1],
          };
        });

        var item = {
          id: this.proposal.sys.id,
          createdAt: this.proposal.sys.createdAt,
          contentType: this.proposal.sys.contentType.sys.id,
          votes: votes,
          fields: fields,
          profile: this.proposal.fields.profile,
        };
        return item;
      } catch (error) {
        console.error(error);
        return [];
      }
    },
    async init() {
      console.log("loading proposal!");

      try {
        var res = await Promise.all([
          axios.get(process.env.VUE_APP_URI + "/treasury/"),
          axios.get(this.uri),
        ]);
        this.treasury = res[0].data.message;
        this.proposal = res[1].data;

        var r = await this.assembleProposalType(
          this.proposal.sys.contentType.sys.id
        );
        this.isVoting = false;
        this.proposalFormat = r;

        console.log("propsoalforamt", r);
        if (_.get(this.proposalFormat, "votes.members.length")) {
          this.members = _.get(this.proposalFormat, "votes.members");
        } else {
          this.members = await this.getMembers();
        }

        this.loaded = true;
        this.$emit("ready");
      } catch (error) {
        console.log("error", error);
      }
    },
  },
  async mounted() {
    this.init();
  },
};
</script>
