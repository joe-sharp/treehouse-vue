<template>
<div>
  <h1>{{ sitename }}</h1>
  <h2><strong class='vue-total'>{{ total }}</strong> Total Points</h2>

  <div class="vue-pieChart group">
    <div class="vue-pie"></div>
    <div class="vue-legend">
      <ul>
        <li v-for="skill in legend" v-bind:key="skill.id" :style="{ 'border-color': getColor(skill.id)}">
          <em>{{ skill.id }}</em><span>{{ skill.value }}</span>
        </li>
      </ul>
    </div>
  </div>
</div>
</template>

<script>
export default {
  name: 'Chart',
  props: ['sitename', 'frontend', 'design', 'backend', 'mobile', 'fundamentals', 'data', 'experimental'],
  data: function () {
    return {
      avatar: '',
      profile: {},
      points: {},
      total: 0,
      legend: [],
      categories: {
        frontend:     0,
        design:       0,
        backend:      0,
        mobile:       0,
        fundamentals: 0,
        data:         0,
        experimental: 0
      },
      color: {
        frontend:     this.frontend,
        design:       this.design,
        backend:      this.backend,
        mobile:       this.mobile,
        fundamentals: this.fundamentals,
        data:         this.data,
        experimental: this.experimental
      }
    }
  },
  methods: {
    filterZeros(hsh) {
      return Object.keys(hsh).filter(key => hsh[key] > 0)
    },
    getKeyByValue(object, value) {
      return Object.keys(object).find(key => object[key] === value);
    },
    descNumericSort(ary) {
      return Object.values(ary).sort((a, b) => a - b ).reverse();
    },
    determineCategory(skill) {
      const check        = skill.toLowerCase()
      const frontend     = ['css', 'html', 'javascript']
      const design       = ['design']
      const backend      = ['apis', 'c#', 'java', 'php', 'python', 'ruby']
      const mobile       = ['android']
      const fundamentals = ['21st century skills', 'business', 'computer science', 'development tools', 'digital literacy', 'learning resources', 'quality assurance', 'security']
      const data         = ['data analysis', 'databases']
      const experimental = ['equity, diversity, and inclusion (edi)', 'go', 'machine learning']
      switch (true) {
        case frontend.includes(check):     return 'frontend';
        case design.includes(check):       return 'design';
        case backend.includes(check):      return 'backend';
        case mobile.includes(check):       return 'mobile';
        case fundamentals.includes(check): return 'fundamentals';
        case data.includes(check):         return 'data';
        case experimental.includes(check): return 'experimental';
      }
    },
    getColor(skill) {
      return this.color[this.determineCategory(skill)]
    },
    createChart(categories) {
      new Chartist.Pie('.vue-pie', {
        series: categories
      }, {
        donut: true,
        donutWidth: 20,
        donutSolid: true,
        startAngle: 0,
        showLabel: false
      });
    },
    colorizeChart() {
      this.setDocStyle('--frontend', this.frontend)
      this.setDocStyle('--design', this.design)
      this.setDocStyle('--backend', this.backend)
      this.setDocStyle('--mobile', this.mobile)
      this.setDocStyle('--fundamentals', this.fundamentals)
      this.setDocStyle('--data', this.data)
      this.setDocStyle('--experimental', this.experimental)
    },
    setDocStyle(prop, val) {
      document.documentElement.style.setProperty(prop, val)
    }
  },
  created: async function () {
    const response = await fetch('https://teamtreehouse.com/joesharp.json');
    const data = await response.json();
    const points = data.points;
    this.profile = data;
    this.total = points.total;
    this.avatar = data.gravatar_url;

    this.filterZeros(points).slice(1).forEach((key) => {
      this.points[key] = points[key]
    });

    const points_desc = Object.values(this.points).sort((a, b) => a - b).reverse();
    points_desc.forEach((value) => {
      let key = this.getKeyByValue(this.points, value)
      let skill = { id: key, value: value }
      this.legend.push(skill);

      this.categories[this.determineCategory(key)] += value
    })

    const categories = Object.values(this.categories)
    this.createChart(categories);
    this.colorizeChart();
  }
}
</script>

<style lang="scss">
  @import url('https://fonts.googleapis.com/css2?family=Source+Code+Pro:wght@400;700&family=Source+Sans+Pro&display=swap');
  $color-background: #f9f9f9;
  $color-primary: #222222;
  :root {
    --frontend: #000;
    --design: #000;
    --backend: #000;
    --mobile: #000;
    --fundamentals: #000;
    --data: #000;
    --experimental: #000;
  }

  * {
    padding: 0;
    margin: 0;
    color: $color-primary;
  }
  .group:after {content:"";display:table;clear:both;}

  ::marker {
    display: none!important;
  }
  .vue-pieChart {
    display: flex;
  }
  div {
    font-family: 'Source Code Pro', monospace;
    background: none;

    a {
      text-decoration: none;
    }

    .team-treehouse {

      h1, h2 {
        text-align: center;
      }
      h2 {
        font-weight: normal;
      }
    }
  }

  .vue-legend {
    width: 13em;
    padding-left: 1em;
    float:left;

    ul {
      list-style-type: none;
      list-style-position: inside;
      font-size: 0.8em;
    }
    li {
      height: 1.25em;
      margin-bottom: 0.7em;
      padding-left: 0.5em;
      border-left: 1.25em solid black;
    }
    em {
      font-family: 'Source Sans Pro', sans-serif;
      font-style: normal;
    }
    span {
      float: right;
      font-weight: bold;
    }
  }

  .ct-series-a .ct-slice-donut-solid{ fill: var(--frontend); }
  .ct-series-b .ct-slice-donut-solid{ fill: var(--design); }
  .ct-series-c .ct-slice-donut-solid{ fill: var(--backend); }
  .ct-series-d .ct-slice-donut-solid{ fill: var(--mobile); }
  .ct-series-e .ct-slice-donut-solid{ fill: var(--fundamentals); }
  .ct-series-f .ct-slice-donut-solid{ fill: var(--data); }
  .ct-series-g .ct-slice-donut-solid{ fill: var(--experimental); }
</style>
