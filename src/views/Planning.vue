<template>
  <div>
  <div class="page-title">
    <h3>{{'Menu_Planning' | localize}}</h3>
    <h4>{{info.bill | currency('BYN')}}</h4>
  </div>

  <Loader v-if="loading" />

  <p class="center" v-else-if="!categories.length">{{'NoCategories' | localize}}. <router-link to="/categories">Добавить новую категорию</router-link></p>

  <section v-else>
    <div v-for="cat of categories" :key="cat.id">
      <p>
        <strong>{{cat.title}}:</strong>
        {{cat.spend | currency('BYN')}} {{ 'Of' | localize }} {{cat.limit | currency('BYN')}}
      </p>
      <div class="progress" v-tooltip.noloc="cat.tooltip">
        <div
            class="determinate"
            :class="[cat.progressColor]"
            :style="{width: cat.progressPercent + '%'}"
        ></div>
      </div>
    </div>
  </section>
</div>
</template>

<script>
import {mapGetters} from 'vuex'
import currencyFilter from '../filters/currency.filter'
import localizeFilter from '../filters/localize.filter'

export default {
  name: 'planning',
   metaInfo() {
    return {
    title: this.$title('Menu_Planning') 
    }
  },
  data: () => ({
    loading: true,
    categories: []
  }),
  computed: {
    ...mapGetters(['info'])
  },
  async mounted() {
    const records = await this.$store.dispatch('fetchRecords')
    const categories = await this.$store.dispatch('fetchCategories')

    this.categories = categories.map(cat => {
      const spend = records
        .filter(r => r.categoryID === cat.id)
        .filter(r => r.type === 'outcome')
        .reduce((total, record) => {
          return total += +record.amount
          }, 0)

      const percent = spend / cat.limit * 100
      const progressPercent = percent > 100 ? 100 : percent
      const progressColor = percent < 60 
        ? 'green'
        : percent < 100 
        ? 'yellow'
        : 'red'

        const tooltipValue = cat.limit - spend
        const tooltip = `${tooltipValue < 0 ? localizeFilter('MoreThan') : localizeFilter('Stayed') } ${currencyFilter(Math.abs(tooltipValue))}`

      return {
        ...cat,
        progressPercent,
        progressColor,
        spend,
        tooltip
      }
    })

    this.loading = false
  }
}
</script>
