<template>
  <header class="z-depth-2">
    <div class="wagerr-logo"></div>

    <ul class="top-nav">
      <router-link tag="li" to="/" exact>
        <i class="navico-wallet"></i>
        <h6>Wallet Home</h6>
      </router-link>

      <router-link tag="li" to="/bet_history">
        <i class="navico-history"></i>
        <h6>Bet History</h6>
      </router-link>

      <router-link tag="li" to="/betting">
        <i class="navico-betting"></i>
        <h6>Betting</h6>
      </router-link>

      <li class="disabled">
        <i class="navico-dice"></i>
        <h6>
          Chain Games
          <br />
          <small>
            (Coming soon!)
          </small>
        </h6>
      </li>

      <router-link tag="li" to="/masternodes">
        <i class="navico-wrench"></i>
        <h6>Masternodes</h6>
      </router-link>

      <router-link tag="li" to="/tools/info">
        <i class="navico-wrench"></i>
        <h6>Tools &amp; Info</h6>
      </router-link>

      <li>
        <el-dropdown trigger="click" placement="bottom-start">
          <div>
            <i class="navico-gear"></i>
            <h6>Settings</h6>
          </div>
          <el-dropdown-menu slot="dropdown" class="topnav-dropdown">
            <el-dropdown-item v-if="!walletEncrypted">
              <a @click="openModal('encrypt-wallet')">
                <i class="icon-file-lock"></i>
                Encrypt Wallet
              </a>
            </el-dropdown-item>
            <el-dropdown-item>
              <a @click="openModal('change-wallet-password')">
                <i class="icon-user-lock"></i>
                Change Password
              </a>
            </el-dropdown-item>
            <el-dropdown-item>
              <a @click="updadteConsoleVisible">
                <i class="icon-cli"></i>
                RPC CLI Tool
              </a>
            </el-dropdown-item>
            <el-dropdown-item>
              <router-link tag="a" to="/preferences" exact>
                <i class="icon-cog"></i>
                Preferences
              </router-link>
            </el-dropdown-item>
          </el-dropdown-menu>
        </el-dropdown>
      </li>

      <!-- Settings Modals -->
      <encrypt-wallet></encrypt-wallet>
      <change-password></change-password>
    </ul>

    <div class="pull-right wagerr-balance">
      <span id="wagerr-total-balance">
        <div class="user-balance-flex">
          <i
            v-if="walletEncrypted"
            :data-target="walletLocked ? 'unlock-wallet-modal' : ''"
            :class="`lock fa ${lockIcon} tooltipped modal-trigger`"
            :data-tooltip="lockText"
            @click="!walletLocked ? lockWallet() : ''"
          >
          </i>
          <el-tooltip placement="left">
            <div slot="content">
              <small>
                Balance: {{ Math.trunc(balance * 100) / 100 }}
                <br />
                Unconfirmed: {{ Math.trunc(pending * 100) / 100 }}
                <br />
                Immature: {{ Math.trunc(immature * 100) / 100 }}
                <br />
                Locked: {{ Math.trunc(lockedBalance * 100) / 100 }}
              </small>
            </div>
            <el-row>
              <el-row>
                {{ walletLoaded ? Math.trunc(balance * 10000) / 10000 : 'Loading...' }}
                <span class="currency">
                  {{ getNetworkType === 'Testnet' ? 'tWGR' : 'WGR' }}
                </span>
              </el-row>
              <el-row>
                <h6>Currently Available</h6>
              </el-row>
            </el-row>
          </el-tooltip>
        </div>
      </span>
    </div>
  </header>
</template>

<script>
import Vuex from 'vuex';
import EncryptWallet from './modals/EncryptWallet.vue';
import ChangePassword from './modals/ChangePassword.vue';

export default {
  name: 'TopNavBar',

  components: {
    EncryptWallet,
    ChangePassword
  },

  data() {
    return {
      intervalHandle: 0
    };
  },

  computed: {
    ...Vuex.mapGetters([
      'balance',
      'immature',
      'lockedBalance',
      'pending',
      'walletLoaded',
      'walletLocked',
      'walletUnlocked',
      'walletUnlockedFully',
      'walletUnlockedOnlyStaking',
      'walletEncrypted',
      'getNetworkType'
    ]),

    lockIcon() {
      if (this.walletLocked) return 'fa-lock';
      if (this.walletUnlockedOnlyStaking) return 'fa-unlock';
      if (this.walletUnlockedFully) return 'fa-lock-open';
    },

    lockText() {
      if (this.walletLocked) return 'Wallet Locked';
      if (this.walletUnlockedOnlyStaking) return 'Wallet Unlocked (only staking)';
      if (this.walletUnlockedFully) return 'Wallet Unlocked';
    }
  },

  mounted() {
    // Initializes modals, dropdown and tooltip
    this.$initMaterialize();

    let isRunning = false;
    this.intervalHandle = setInterval(async () => {
      if (!isRunning) {
        isRunning = true;
        await this.walletInfo();
        await this.walletExtendedBalance();
        isRunning = false;
      }
    }, 3000);
  },

  beforeDestroy() {
    clearInterval(this.intervalHandle);
  },

  methods: {
    ...Vuex.mapActions([
      'lockWallet',
      'updadteConsoleVisible',
      'walletExtendedBalance',
      'walletInfo'
    ]),

    openModal(modalId) {
      const modal = document.getElementById(modalId);
      M.Modal.getInstance(modal).open();
    }
  }
};
</script>
<style lang="scss">
@import '../assets/scss/element-ui';

// Dropdown list
@include dropdown(topnav-dropdown, #212529);
</style>

<style scoped lang="scss">
@import '../assets/scss/_variables';

.user-balance-flex {
  display: flex;

  i.lock {
    margin-left: 0;
    font-size: 1.2em;
    margin-right: 10px;
    margin-top: 2px;

    &.fa-lock {
      color: $wagerr-red;
    }
    &.fa-unlock {
      color: #aaa;
    }
    &.fa-lock-open {
      color: white;
      font-size: 1em;
      margin-top: 4px;
    }
  }
}

// Dropdown button
.el-dropdown {
  width: 100%;
  color: white;
  height: 100%;

  // - Fixes behaviour of dropdown div trigger
  div.el-dropdown-selfdefine {
    width: 100%;
    height: 100%;
    position: absolute;
    outline: none;
  }
}

.disabled {
  color: rgba(255,255,255,0.5);
  cursor: default;
}

.disabled i {
  cursor: default;
}

.disabled:hover {
  background-color: transparent;
}
</style>
