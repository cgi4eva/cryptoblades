<template>
  <div class="body main-font">
    <b-tabs justified>
      <b-tab title="Weapons">
        <div class="blank-slate" v-if="ownWeapons.length === 0">
          You do not currently have any weapons.
          <br>
          You can forge one by clicking the button below.
          <br>
          <br>
          <big-button
            class="button"
            :mainText="`Forge sword for ${forgeCost} SKILL`"
            @click="onForgeWeapon"
          />
        </div>

        <div class="row mt-3" v-if="ownWeapons.length > 0">
          <div class="col">
            <div class="d-flex justify-content-space-between">
              <h1>Weapons ({{ ownWeapons.length }})</h1>

              <div class="d-flex justify-content-flex-end ml-auto">
                <b-button
                  variant="primary"
                  v-if="reforgeWeaponId !== null && ownWeapons.length > 0"
                  @click="showReforge = true"
                  tagname="reforge_weapon"
                  v-tooltip="'Burn weapons to buff selected weapon'">
                  Reforge
                </b-button>

                <b-button
                  variant="primary"
                  class="ml-3"
                  @click="onForgeWeapon"
                  :disabled="disableForge"
                  v-tooltip="'Forge new weapon'">
                  <span v-if="disableForge">
                    Cooling forge...
                  </span>

                  <span v-if="!disableForge" class="gtag-link-others" tagname="forge_weapon">
                    Forge x1 ({{ forgeCost }} SKILL) <i class="fas fa-plus"></i>
                  </span>
                </b-button>

                  <b-button
                  variant="primary"
                  class="ml-3"
                  @click="onForgeWeaponx10()"
                  :disabled="disableForge"
                  v-tooltip="'Forge new weapon'">
                  <span v-if="disableForge">
                    Cooling forge...
                  </span>

                  <span v-if="!disableForge" class="gtag-link-others" tagname="forge_weapon">
                    x10 ({{ forgeCost*10 }} SKILL) <i class="fas fa-plus"></i>
                  </span>
                </b-button>

                <b-icon-question-circle class="centered-icon" scale="1.5"
                  v-on:click="onShowForgeDetails" v-tooltip.bottom="'Click for forge percentages'"/>

                <b-modal hide-footer ref="forge-details-modal" title="Forge Percentages">
                  <div>
                    5+ star @ 1% chance. Estimated cost {{Number.parseFloat(forgeCost * (1/0.01)).toFixed(2)}} SKILL.
                  </div>
                  <div>
                    4+ star @ 6% chance. Estimated cost {{Number.parseFloat(forgeCost * (1/0.06)).toFixed(2)}} SKILL.
                  </div>
                  <div>
                    3+ star @ 21% chance. Estimated cost {{Number.parseFloat(forgeCost * (1/0.21)).toFixed(2)}} SKILL.
                  </div>
                  <div>
                    2+ star @ 56% chance. Estimated cost {{Number.parseFloat(forgeCost * (1/0.56)).toFixed(2)}} SKILL.
                  </div>
                  <div>
                    1+ star @ 100% chance.
                  </div>
                </b-modal>
              </div>
            </div>

            <weapon-grid v-model="reforgeWeaponId" />
            <b-modal size="xl" class="centered-modal " ref="new-weapons" ok-only>
                        <template #modal-header>
                            <div v-if="!spin" class="new-weapon-header-text text-center">
                                  <strong>Forging Results</strong>
                            </div>
                            <div v-if="spin" class="new-weapon-header-text text-center">
                                  <strong>Be patient, the irons are hot...</strong>
                            </div>
                        </template>
                        <div class="text-center">
                          <b-spinner v-if="spin" type="grow" label="Loading..."></b-spinner>
                          <b-spinner v-if="spin" type="grow" label="Loading..."></b-spinner>
                          <b-spinner v-if="spin" type="grow" label="Loading..."></b-spinner>
                        </div>
                        <weapon-grid v-if="!spin" :showGivenWeaponIds="true" :weaponIds="newForged" :newWeapon="true"/>

                        <template #modal-footer>
                        </template>
                    </b-modal>
          </div>

          <div class="col-6" v-if="showReforge">
            <div class="d-flex justify-content-space-between">
              <h1>Choose Burn Weapon</h1>

              <div class="d-flex justify-content-flex-end ml-auto">
                <b-button
                  variant="primary"
                  class="ml-3"
                  tagname="confirm_forge_weapon"
                  @click="showReforgeConfirmation"
                  :disabled="canReforge"
                  v-tooltip="'Forge new weapon'">
                  Confirm Reforge ({{ reforgeCost }} SKILL)
                </b-button>
              </div>
            </div>

            <b-modal class="centered-modal" ref="reforge-confirmation-modal" title="Reforge Confirmation"
              @ok="onReforgeWeapon">
              <div class="text-center" v-text="'Are you sure you want to reforge with this weapon?'" />
              <div class="weapon" :hidden="burnWeaponId == null">
                <div class="above-wrapper" v-if="$slots.above || $scopedSlots.above">
                  <slot name="above" :weapon="getWeaponToBurn()"></slot>
                </div>
                <div class="weapon-icon-wrapper">
                  <weapon-icon class="weapon-icon" :weapon="getWeaponToBurn()" />
                </div>
              </div>
              <div class="text-center" :hidden="burnWeaponId == null || !isWeaponRare()">
                <b-icon icon="exclamation-circle" variant="danger" /> This is a rare weapon!
              </div>
              <div class="text-center" :hidden="burnWeaponId == null || !isWeaponReforged()">
                <b-icon icon="exclamation-circle" variant="danger" />
                This item has been previously reforged and only half of each LBs will carry over!
              </div>
            </b-modal>

            <weapon-grid v-model="burnWeaponId" :ignore="reforgeWeaponId" :showReforgedWeaponsDefVal="false" :showFavoriteWeaponsDefVal="false" />
          </div>
        </div>
      </b-tab>
      <b-tab>
        <template #title>
          Shields <b-icon-question-circle class="centered-icon" scale="0.8" v-tooltip.bottom="`You can buy shield in Skill shop tab in the market!`"/>
        </template>
        <nft-list :nfts="ownNfts"/>
      </b-tab>
    </b-tabs>
  </div>
</template>

<script>
import BN from 'bignumber.js';
import WeaponGrid from '../components/smart/WeaponGrid.vue';
import BigButton from '../components/BigButton.vue';
import { mapActions, mapGetters, mapState } from 'vuex';
import WeaponIcon from '../components/WeaponIcon.vue';
import { BModal } from 'bootstrap-vue';
import NftList from '@/components/smart/NftList.vue';


export default {


  data() {
    return {
      showReforge: false,
      reforgeWeaponId: null,
      burnWeaponId: null,
      forgeCost: 0,
      reforgeCost: 0,
      disableForge: false,
      forgeMultiplier: 10,
      newForged: [],
      currentListofWeapons: [],
      x10Forge: false,
      x1Forge: false,
      onError: false,
      spin: false,
    };
  },

  computed: {
    ...mapState(['defaultAccount','ownedWeaponIds','ownedShieldIds']),
    ...mapGetters(['contracts', 'ownWeapons', 'ownShields']),

    canReforge() {
      return (
        this.reforgeWeaponId === null ||
        this.burnWeaponId === null ||
        this.reforgeWeaponId === this.burnWeaponId
      );
    },

    ownNfts() {
      const ownNfts = [];

      // get various types of nfts and push to ownNfts list
      const shieldsIdTypes = this.ownedShieldIds.map(id => { return { nftId: id, nftType: 'shield'}; });

      ownNfts.push(shieldsIdTypes);
      return shieldsIdTypes;
    }
  },

  watch: {
    reforgeWeaponId() {
      this.showReforge = false;
      this.burnWeaponId = null;
    }
  },

  async created() {
    const forgeCost = await this.contracts.CryptoBlades.methods.mintWeaponFee().call({ from: this.defaultAccount });
    const skillForgeCost = await this.contracts.CryptoBlades.methods.usdToSkill(forgeCost).call();
    this.forgeCost = BN(skillForgeCost).div(BN(10).pow(18)).toFixed(4);

    const reforgeCost = await this.contracts.CryptoBlades.methods.reforgeWeaponFee().call({ from: this.defaultAccount });
    const skillReforgeCost = await this.contracts.CryptoBlades.methods.usdToSkill(reforgeCost).call();
    this.reforgeCost = BN(skillReforgeCost).div(BN(10).pow(18)).toFixed(4);
  },

  methods: {
    ...mapActions(['mintWeapon', 'reforgeWeapon', 'mintWeaponN']),

    async onForgeWeapon() {
      if(this.disableForge) return;

      this.getCurrentListofWeapons();
      this.onError = false;
      this.x1Forge = true;
      this.disableForge = true;

      // Incase the network or mm are having issues, after 1 min we reshow
      const failbackTimeout = setTimeout(() => {
        this.disableForge = false;
      }, 60000);

      try {
        await this.mintWeapon();

        this.viewNewWeapons(1);

      } catch (e) {
        console.error(e);
        this.onError = true;
        this.$dialog.notify.error('Could not forge sword: insuffucient funds or transaction denied.');
      } finally {
        clearTimeout(failbackTimeout);
        this.disableForge = false;
      }
    },

    async onForgeWeaponx10(){
      if(this.disableForge) return;
      this.disableForge = true;

      this.getCurrentListofWeapons();
      this.onError = false;
      this.x10Forge = true;

      // Incase the network or mm are having issues, after 1 min we reshow
      const failbackTimeout = setTimeout(() => {
        this.disableForge = false;
      }, 60000);

      try {
        await this.mintWeaponN({num: this.forgeMultiplier});

        this.viewNewWeapons(this.forgeMultiplier);

      } catch (e) {
        console.error(e);
        this.onError = true;
        this.$dialog.notify.error('Could not forge sword: insuffucient funds or transaction denied.');
      } finally {
        clearTimeout(failbackTimeout);
        this.disableForge = false;
      }
    },
    onShowForgeDetails() {
      this.$refs['forge-details-modal'].show();
    },

    showReforgeConfirmation() {
      this.$refs['reforge-confirmation-modal'].show();
    },

    isWeaponRare() {
      return this.getWeaponToBurn().stars >= 3;
    },

    isWeaponReforged() {
      return this.getWeaponToBurn().bonusPower > 0;
    },

    getWeaponToBurn() {
      return this.ownWeapons.find(x => x.id === this.burnWeaponId);
    },

    getCurrentListofWeapons(){
      this.ownedWeaponIds.forEach(x => {
        this.currentListofWeapons.push(x);
      });
    },

    viewNewWeapons(newWeaponCount = 1){
      this.newForged = this.ownedWeaponIds.slice(-newWeaponCount);

      // eslint-disable-next-line no-constant-condition
      if (this.newForged.length > 0 && !this.onError){
        this.spin = true;
        this.$refs['new-weapons'].show();

        setTimeout(() => {
          this.spin = false;
        }, 10000);
      }


    },

    async onReforgeWeapon() {
      try {
        await this.reforgeWeapon({
          burnWeaponId: this.burnWeaponId,
          reforgeWeaponId: this.reforgeWeaponId,
        });

        this.burnWeaponId = null;
      } catch (e) {
        console.error(e);
        this.$dialog.notify.error('Could not forge sword: insuffucient funds or transaction denied.');
      }
    }
  },

  components: {
    WeaponGrid,
    BigButton,
    WeaponIcon,
    BModal,
    NftList,
  },
};
</script>

<style scoped>

.new-weapon-header-text{
   color: #9e8a57;
  font-size: 34px;
}

.button-row {
  margin-top: 1em;
  display: flex;
}

.mint-weapon {
  height: 2.5rem;
  width: 14rem;
  font-size: 1.3rem;
  color: rgba(255, 255, 255, 0.6);
  background: none;
  border: none;
  border-radius: 0.1em;
  margin-right: 1em;
  float: right;
}

.weapon-container {
  display: flex;
  flex-direction: row;
  width: 100%;
}

.sub-container {
  flex: 1;
}

.weapon {
  width: 12em;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 5px;
  cursor: pointer;
  margin-left: auto;
  margin-right: auto;
}

.centered-modal {
  justify-content: center;
}

.centered-icon {
  align-self: center;
  margin-left: 5px;
}

</style>
