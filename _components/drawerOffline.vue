<template>
  <div id="drawerOffline" class="q-pa-md">
    <!-- ===== Header ===== -->
    <div class="row justify-between items-center">
      <div class="text-subtitle1 row items-center">
        <q-icon name="fa fa-cloud-upload" color="primary" size="20px" class="q-mr-sm" />
        <label>{{ $tr('isite.cms.label.offlineRequests', { capitalize: true }) }}</label>
      </div>
      <!-- Close icon -->
      <q-icon name="fas fa-times" color="blue-grey" size="20px" class="cursor-pointer"
        @click="eventBus.emit('toggleMasterDrawer', 'offline')" />
    </div>
    <!--Separator-->
    <q-separator class="q-my-md" />
    <requestList />
  </div>
  <dataSynchronizationModal v-if="isOpenModal">
    <template v-slot:default>
      <requestList />
    </template>
  </dataSynchronizationModal>
</template>

<script>
import { eventBus } from 'src/plugins/utils'
import { preloadData } from '../_plugins/handleModuleCalls'
import requestList from './requestList.vue';
import dataSynchronizationModal from './dataSynchronizationModal.vue';


export default {
  name: "drawerOffline",
  props: {},
  components: {
    requestList,
    dataSynchronizationModal
  },
  watch: {
    isAppOffline: {
      handler(newValue) {
        if (!newValue && this.pendingRequests) {
          this.$store.dispatch('qofflineMaster/OPEN_MODAL_SYNC')
        }
        if (newValue) {
          this.$store.dispatch('qofflineMaster/CLOSE_MODAL_SYNC')
        }
      },
      deep: true
    },
    numberPendingRequests: {
      handler(newValue) {
        if (navigator.onLine && newValue > 0) {
          this.$store.dispatch('qofflineMaster/OPEN_MODAL_SYNC')
        }
      },
      deep: true
    },
  },
  beforeUnmount() {
    eventBus.off('header.badge.manage');
  },
  mounted() {
    this.$nextTick(async () => {
      this.$store.dispatch(
        'qofflineMaster/OFFLINE_REQUESTS',
        { userId: this.$store.state.quserAuth.userId }
      )

      if (!this.$store.state.qofflineMaster.dataPreloadDone) {
        await preloadData()
        this.$store.dispatch('qofflineMaster/PRELOADED_DATA')
      }

    });
  },
  computed: {
    requests() {
      return this.$store.state.qofflineMaster.requestsReversed
    },
    isAppOffline() {
      return this.$store.state.qofflineMaster.isAppOffline
    },
    numberPendingRequests() {
      return this.$store.state.qofflineMaster.pendingRequests
    },
    pendingRequests() {
      return this.numberPendingRequests > 0
    },
    isOpenModal() {
      return this.$store.state.qofflineMaster.isOpenModalSync
    }
  },
  data() {
    return {
      refreshIntervalId: null,
      eventBus,
    }
  },
  methods: {
    getTitle(request) {
      if (request?.requestData?.body) {
        const body = JSON.parse(request.requestData.body)
        const attributes = body?.attributes

        return {
          id: attributes?.id || '',
          titleOffline: attributes?.titleOffline || attributes?.title_offline || ''
        }
      }
    },
  },
}
</script>
