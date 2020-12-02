<template>
<div>
  <h1>{{ sitename }}</h1>
  <h2><strong class='vue-total'>{{ total }}</strong> Total Points</h2>

  <div class="vue-pieChart group">
    <div class="vue-pie"></div>
    <div class="vue-legend">
      <ul>
        <li v-for="skill in legend" v-key="skill.id" :style="{ 'border-color': getColor(skill.id)}">
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
  props: ['sitename'],
  data: function () {
    return {
      profile: {},
      points: {},
      total: 0,
      legend: [],
      categories: {
        frontend: 0,
        design: 0,
        backend: 0,
        mobile: 0,
        fundamentals: 0,
        data: 0,
        experimental: 0
      },
      color: {
        frontend: '#3659A2',
        design: '#4A4290',
        backend: '#008297',
        mobile: '#30826C',
        fundamentals: '#9B3B5A',
        data: '#9F4B84',
        experimental: '#733A88'
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
      const check = skill.toLowerCase()
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
    }
  },
  /**
   * [Main]
   */
  async created() {
    // fetch('https://teamtreehouse.com/joesharp.json').then(response => response.json()).then(data => this.points = data.points)
    const response = await fetch('https://teamtreehouse.com/joesharp.json');
    const data = await response.json();
    const points = data.points;
    this.profile = data;

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

    this.total = points.total;
  }
}
</script>

<style lang="scss" scoped>
  @import url('https://fonts.googleapis.com/css?family=Open+Sans:400,700');
  // properties
  $color-background: #f9f9f9;
  $color-primary: #222222;

  // resets
  * {
    padding: 0;
    margin: 0;
    color: $color-primary;
  }

  // clearfix
  .group:after {content:"";display:table;clear:both;}

  // general
  ::marker {
    display: none!important;
  }
  .vue-pieChart {
    display: flex;
  }
  body {
    font-family: "Open Sans", Arial;
    background: $color-background;

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
  // legend
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
      font-style: normal;
    }
    span {
      float: right;
      font-weight: bold;
    }
  }
</style>
