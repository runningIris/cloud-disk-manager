<template>
  <div class="file-list-topbar">
    <div
      class="folder-button"
      title="选择目录"
      @click="handleFolderSelect"
    >
      <span class="folder-button-icon">
        <svg-icon icon-class="dir" />
      </span>
      <span>目录</span>
    </div>
    <div
      v-show="dir.split(sep).filter(i=>i !== '').length > 1"
      class="folder-button"
      title="上一级"
      @click="handleChangeUp"
    >
      <span class="folder-button-icon">
        <svg-icon icon-class="up" />
      </span>
    </div>
    <div
      v-show="historys[dir] && historys[dir].from"
      class="folder-button"
      title="返回"
      @click="handleChangeDir(historys[dir].from)"
    >
      <span class="folder-button-icon">
        <svg-icon icon-class="back-folder" />
      </span>
      <span>后退</span>
    </div>
    <div
      v-show="historys[dir] && historys[dir].to"
      class="folder-button"
      title="前进"
      @click="handleChangeDir(historys[dir].to)"
    >
      <span class="folder-button-icon">
        <svg-icon icon-class="ahead-folder" />
      </span>
      <span>前进</span>
    </div>

    <div
      class="folder-button"
      title="刷新（F5）"
      @click="$emit('refresh')"
    >
      <span class="folder-button-icon">
        <svg-icon
          :class="listingDir === 1 ? 'rorate-animate' : ''"
          icon-class="refresh"
        />
      </span>
    </div>
    <file-dir-path-bar
      :dir="dir"
      @changeDir="(targetDir)=>handleChangeDir(targetDir)"
    />
  </div>
</template>

<script>
import fileDirPathBar from './fileDirPathBar.vue'

const PATH = require('path')

export default {
  components: {
    fileDirPathBar
  },
  props: {
    dir: {
      type: String,
      default: ''
    },
    listingDir: {
      type: Number,
      default: 0
    }
  },
  data () {
    return {
      historys: {},
      sep: PATH.sep

    }
  },
  watch: {
    // 目录改变则及时记录历史
    dir (currentDir, beforeDir) {
      this.historysHandler(currentDir, beforeDir)
    }
  },
  methods: {
    // 跳至上级目录
    handleChangeUp () {
      const targetDir = this.dir.split(this.sep).slice(0, -1).join(this.sep)
      this.handleChangeDir(targetDir.includes(this.sep) ? targetDir : targetDir + this.sep)
    },

    // 改变目录
    handleChangeDir (dir) {
      this.$emit('changeDir', dir)
    },

    // 目录变更历史记录处理，用于前进及后退
    historysHandler (currentDir, beforeDir) {
      if (beforeDir) {
        if (!this.historys[beforeDir]) {
          this.historys[beforeDir] = { to: currentDir }
        } else if (this.historys[beforeDir].from !== currentDir) {
          this.historys[beforeDir].to = currentDir
        }

        if (!this.historys[currentDir]) {
          this.historys[currentDir] = { from: beforeDir }
        } else if (this.historys[currentDir].to !== beforeDir) {
          this.historys[currentDir].from = beforeDir
        }
      }
      console.log(this.historys, 'dirChangeHistorys')
    },

    // 选择目录，等效于同步方法
    async handleFolderSelect () {
      const selection = await this.$bus.dialog.showOpenDialog(this.$bus.win, {
        properties: [
          'openDirectory',
          'showHiddenFiles',
          'treatPackageAsDirectory'
        ],
        message: '请选择您要打开的目录'
      })
      if (!selection.canceled && selection.filePaths[0]) {
        this.handleChangeDir(selection.filePaths[0])
      }
    }
  }
}
</script>

<style lang="less" scoped>
@import "../../themes/light.less";
.folder-button {
  display: flex;
  justify-content: center;
  align-items: center;
  white-space: nowrap;
  padding: 0 10px;
  background-color: #f1f2f6;
  border-radius: 4px;
  transition: 0.2s ease;
  font-size: 13px;
  margin-left: 8px;
  cursor: pointer;
  height: 36px;
  color: #283593;
}

.folder-button:hover {
  filter: brightness(0.9);
}

.folder-button:active {
  filter: brightness(0.8);
}

.folder-button-icon {
    padding: 0 5px;
}

.file-list-topbar {
  padding: 0 5px;
  display: flex;
  align-items: center;
  //border-bottom: 1px dashed #d5d8e3;
  height: 60px;
  min-height: 60px;
  font-size: 12px;
  color: #616161;
  box-shadow: 0 0px 4px rgba(55, 55, 77, 0.1);
}

.rorate-animate {
  animation: circle 0.6s infinite linear;
}

@keyframes circle {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}
</style>
