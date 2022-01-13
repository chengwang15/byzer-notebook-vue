<template>
  <div class="set-demo">
    <div v-if="userInfo.is_admin && !demoList.includes(activeNotebook.id)" class="btn">
      <span class="hasEvent" @click="handleOperateDemo('publish')">
        {{ $t('notebook.setDemo') }}
      </span>
    </div>
    <el-dropdown
      v-if="userInfo.is_admin && demoList.includes(activeNotebook.id)"
      class="btn" trigger="hover"
      @command="handleOperateDemo"
    >
      <span class="drop-text hasEvent">
        {{ $t('notebook.demoOn') }}
        <i class="el-ksd-icon-arrow_down_22 font-22"></i>
      </span>
      <el-dropdown-menu slot="dropdown">
        <el-dropdown-item command="update">
          {{ $t('notebook.updateDemo') }}
        </el-dropdown-item>
        <el-dropdown-item command="remove">
          {{ $t('notebook.removeDemo') }}
        </el-dropdown-item>
      </el-dropdown-menu>
    </el-dropdown>
  </div>
</template>

<script>
import Vue from 'vue'
import { mapActions, mapState } from 'vuex'
import { Component } from 'vue-property-decorator'
import { FileSuffixMap } from '@/config'

@Component({
  props: {
    userInfo: {
      default: () => ({}),
      type: Object
    },
    activeNotebook: {
      default: () => ({}),
      type: Object
    }
  },
  computed: {
    ...mapState({
      demoList: state => state.notebook.demoList
    })
  },
  methods: {
    ...mapActions({
      setDemo: 'SET_DEMO',
      removeDemo: 'REMOVE_DEMO'
    })
  }
})
export default class SetDemo extends Vue {
  /**
   * @param {*} command publish/update/remove
   * @Date: 2022-01-12 16:31:55
   */
  async handleOperateDemo (command) {
    const isRemove = command === 'remove'
    const text = this.$t('notebook.demoText', {
      type: this.$t('notebook.' + command + 'Demo').toLowerCase(),
      fileName:
        this.activeNotebook?.name + FileSuffixMap[this.activeNotebook?.type]
    })
    const title = this.$t('notebook.' + command + 'Demo')

    this.$confirm(text, title, {
      confirmButtonText: title,
      confirmButtonClass: isRemove ? 'el-button--danger' : '',
      cancelButtonText: this.$t('cancel'),
      type: isRemove ? 'error' : 'warning',
      customClass: 'centerButton'
    })
      .then(async () => {
        const params = {
          entity_id: this.activeNotebook?.id,
          entity_type: this.activeNotebook?.type
        }

        let res = {};
        if (isRemove) {
          res = await this.removeDemo(params)
        } else {
          res = await this.setDemo(params)
        }

        if (res?.data === 'success') {
          this.$message.success(this.$t('notebook.' + command + 'Success'))
          this.$emit('operateDemoSuccess')
        }
      })
  }
}
</script>
<style lang="scss">
@import '@/assets/css/variable.scss';
.set-demo {
  display: inline-flex;
  .btn {
    height: 22px;
    margin-right: 15px;
    line-height: 22px;
    .drop-text {
      color: $--color-black;
      i {
        vertical-align: middle;
      }
    }
  }
}
</style>
