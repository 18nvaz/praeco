<template>
  <div>
    <h1>{{ type }}/{{ path }}</h1>

    <el-button type="primary" plain @click="addFolder">Add folder</el-button>

    <el-button type="danger" plain @click="deleteFolder">Delete folder...</el-button>
  </div>
</template>

<script>
export default {
  props: ['type', 'path'],
  methods: {
    addFolder() {
      this.$prompt('Name', 'Add folder', {
        confirmButtonText: 'OK',
        cancelButtonText: 'Cancel'
      })
        .then(async ({ value }) => {
          await this.$store.dispatch('configs/createFolder', {
            path: `${this.path}/${value}`,
            type: this.type
          });
          this.$router.push({
            path: `/folders/${this.type}/${this.path}/${value}`,
            query: { refreshTree: true }
          });
        })
        .catch(() => {});
    },
    async deleteFolder() {
      try {
        let res = await this.$store.dispatch('configs/deleteFolder', {
          path: this.path,
          type: this.type
        });

        if (res.deleted) {
          this.$message.success('Deleted folder successfully');
          let path = res.path.split('/');
          path.pop();
          path = path.join('/');

          if (path.length) {
            this.$router.push({
              path: `/folders/${this.type}/${path}`,
              query: { refreshTree: true }
            });
          } else {
            this.$router.push({ path: `/${this.type}`, query: { refreshTree: true } });
          }
        }
      } catch (error) {
        this.$message.error('Error deleting folder, is it empty?');
      }
    }
  }
};
</script>
