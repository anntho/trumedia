<template>
  <n-space vertical :size="8">
    <n-data-table
      :columns="columns"
      :data="data"
      :pagination="pagination"
      size="small"
    />
  </n-space>
</template>

<script>
const columns = [
  { title: 'Date', key: 'date', defaultSortOrder: 'ascend',
    sorter: (row1, row2) => new Date(row1.date) - new Date(row2.date),
  },
  { title: 'Week', key: 'week', sorter: 'default' },
  { title: 'Opp', key: 'opp', sorter: 'default' },
  { title: 'Cmp', key: 'cmp', sorter: 'default' },
  { title: 'Att', key: 'att', sorter: 'default' },
  { title: 'Cmp%', key: 'cmppct', sorter: 'default' },
  { title: 'Avg', key: 'avg', sorter: 'default' },
  { title: 'PsYds', key: 'psyds', sorter: 'default' },
  { title: 'PsTD',key: 'pstd', sorter: 'default' },
  { title: 'Rush', key: 'rush', sorter: 'default' },
  { title: 'RshYds', key: 'rshyds', sorter: 'default' },
  { title: 'RshTD', key: 'rshtd', sorter: 'default' },
  { title: 'TD', key: 'td', sorter: 'default' },
  { title: 'Int', key: 'int', sorter: 'default' },
  { title: 'Sack', key: 'sack', sorter: 'default' },
]

export default {
  props: {
    dataProp: {
      type: Object,
      required: true,
    },
  },
  watch: {
    dataProp: {
      deep: true,
      handler(newVal) {
        this.data = newVal
      },
    },
  },
  data() {
    return {
      data: this.dataProp,
      pagination: { pageSize: 5 },
      columns,
    }
  },
}
</script>