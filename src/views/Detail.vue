<template>
  <div>
    <Loader v-if="loading" />
  <div v-else-if="record">
    <div class="breadcrumb-wrap">
      <router-link to="/history" class="breadcrumb">{{ 'Menu_History' | localize }}</router-link>
      <a @click.prevent class="breadcrumb">
        {{ record.type === 'income' ? 'Income' : 'Outcome' | localize }}
      </a>
    </div>
    <div class="row">
      <div class="col s12 m6">
        <div class="card" :class="{
          'green' : record.type === 'income',
          'red' : record.type === 'outcome'
          }">
          <div class="card-content white-text">
            <p>{{ 'Description' | localize}}: {{ record.description }}</p>
            <p>{{ 'Total' | localize }}: {{ record.amount | currency}}</p>
            <p>{{ 'Category' | localize }}: {{ record.categoryName }}</p>

            <small>{{ new Date(record.date) | date('datetime') }}</small>
          </div>
        </div>
      </div>
    </div>
  </div>
  <p class="center" v-else> Запись с id= <strong>{{ $route.params.id }}</strong> не найдена</p>
</div>
</template>

<script>
export default {
  name: 'detail',
   metaInfo() {
    return {
    title: this.$title('DetailTitle') 
    }
  },
  data: () => ({
    record : null,
    loading: true
  }),
  async mounted() {
    const id = this.$route.params.id
    const record = await this.$store.dispatch('fetchRecordByID', id)
    const category = await this.$store.dispatch('fetchCategoryByID', record.categoryID)

    this.record = {
      ...record,
      categoryName: category.title
    }

    this.loading = false
  }
}
</script>
