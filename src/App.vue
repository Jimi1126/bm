<template>
  <main>
    <el-input
      v-model="searchText"
      autofocus
      clearable
      placeholder="输入过滤关键词"
      @input="onQueryChanged"
      @keyup.enter="onQueryChanged"
    >
    </el-input>
    <el-tree-v2
      ref="v2treeRef"
      :props="props"
      :height="320"
      :default-expanded-keys="defaultExpandedKeys"
      :filter-method="filterMethod"
      ><template #default="{ data }">
        <div v-if="data.url" class="tree-node">
          <el-link :href="data.url" target="_blank"
            ><el-text truncated>{{ data.title }}</el-text></el-link
          >
          <el-link :href="data.url"> 当前 </el-link>
          <el-link :href="data.url" target="_blank"> 新页 </el-link>
        </div>
        <el-text v-else-if="data.title" truncated>{{ data.title }}</el-text>
      </template>
    </el-tree-v2>
  </main>
</template>

<script setup lang="ts">
const props = {
  value: 'id',
  label: 'title',
  children: 'children'
}
const v2treeRef = ref(null)
const searchText = ref('')
let defaultExpandedKeys = ref([])
function onQueryChanged() {
  v2treeRef.value?.filter(searchText.value)
}
const filterMethod = (query: any, node: any) => {
  return node.title && node.title.toLowerCase().includes(query.toLowerCase())
}

onMounted(() => {
  chrome.bookmarks.getTree((markNodes: any) => {
    v2treeRef.value?.setData(markNodes[0].children)
    markNodes[0].children.forEach((n) => {
      defaultExpandedKeys.value.push(n.id)
      if (n.children) {
        n.children.forEach((nn) => defaultExpandedKeys.value.push(nn.id))
      }
    })
  })
})
</script>

<style scoped>
main {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  margin: 0 auto;
  padding: 2rem;
}
.tree-node {
  display: grid;
  grid-template-columns: auto max-content max-content;
  width: 100%;
  gap: 12px;

  a {
    width: 100%;
    justify-content: start;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }
}
</style>
