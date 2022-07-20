<template>
  <q-page class="main">
    <div class="q-pa-md">
      <div class="page">

        <!-- Лайаут -->

          <div class="nav_bar">
            <img src="../../images/header.png" alt="header" class="header_image">

            <!-- Список сохранения графиков -->

            <div class="list_block">
              <q-btn 
                round
                icon="inbox" 
                class="list_button"
                size="lg"
                color="white"
                text-color="green"
                @click="list()"
              ></q-btn>
              <div class="list">
                <div class="list-text">
                  <strong>
                    Список оформленных кредитов
                  </strong>
                </div>
                <q-list bordered separator>
                  <q-item
                    clickable 
                    v-ripple 
                    active-class="bg-teal-1 text-grey-8" 
                    v-for="credit in credits"
                  >
                    <q-item-section avatar>
                      <q-icon name="fas fa-calendar-check" />
                    </q-item-section>
                    <q-item-section class="credit"> 
                      <strong>
                        {{ credit }} 
                      </strong>
                    </q-item-section>
                    <q-item-section side class="section"> {{ date }}</q-item-section>
                  </q-item>
                </q-list>
              </div>
            </div>
          </div>
        </div>

        <!-- Блок отправки данных -->

        <div class="main_p">
          <div class="anime_load">
            <div class="graf_hidden"></div>
          </div>
          <div class="anime_load">
            <div class='choise_block'>
              <q-input
                dark standout bottom-slots label-slot
                type='number' min='0' max='7320' 
                v-model="days" 
                @change="setStat()"
              >
                <template v-slot:label>
                  <strong style="font-family: 'Balsamiq Sans', cursive;">
                    Срок в днях
                  </strong>
                </template>
              </q-input>

              <q-input
                dark standout bottom-slots label-slot
                type='number' min='500' 
                v-model="sum" 
                @change="setStat()"
              >
                <template v-slot:label>
                  <strong style="font-family: 'Balsamiq Sans', cursive;">
                    Сумма кредита
                  </strong>
                </template>
              </q-input>

              <q-input
                dark standout bottom-slots label-slot
                type='number' min='0' max="50"
                v-model="percent" 
                @change="setStat()"
              >
                <template v-slot:label>
                  <strong style="font-family: 'Balsamiq Sans', cursive;">
                    Проценты
                  </strong>
                </template>
              </q-input>

              <div>

                <!-- Выбор даты -->

                <q-btn 
                  icon="event" 
                  round 
                  color="primary"
                  style="
                    width: 56px;
                    height: 56px;
                    margin: 20px 0px 0px 0px
                ">
                  <q-popup-proxy @before-show="updateProxy" transition-show="scale" transition-hide="scale">
                    <q-date v-model="time">
                      <div class="row items-center justify-end q-gutter-sm">
                        <q-btn label="Cancel" color="primary" flat v-close-popup />
                        <q-btn label="OK" color="primary" flat @click="save" v-close-popup />
                      </div>
                    </q-date>
                  </q-popup-proxy>
                </q-btn>

                <q-btn
                  @click="testing()"
                  style="margin: 20px 0px 0px 20px"
                  icon-right="send"
                  color='green'
                  label-slot
                  size="lg"
                >
                  <template class="request_text">
                    <strong>
                      Запрос⠀
                    </strong>
                  </template>
                </q-btn>
              </div>
            </div>
          </div>
        </div>

        
        <!-- График -->

        <div class="graf">
            <div
              class="col-lg-6 col-md-6 col-sm-12 col-xs-12" 
              style="width: 1000px;"
            >
              <Chart ref="Chart" :series="series" :xAxis="xAxis"
              v-if="this.graf_status"/>
            </div>
          </div>

        <div class="tables_hidden">
        </div>

        <!-- Таблицы -->

        <div class="tables" v-if="this.graf_status">
          <q-table
            virtual-scroll-item-size="48"
            class="my-sticky-header-column-table"
            table-layout="fixed"
            style="width: 600px;"
            title="Аннуитентный платеж"
            :data="rows[0]"
            :columns="columns[0]"
            row-key="name"
            dark
            color="amber"
            :rows-per-page-options="[0]"
          />

          <q-table
            class="my-sticky-header-column-table"
            style="width: 600px"
            title="Дифференциированный платеж"
            :data="rows[1]"
            :columns="columns[1]"
            row-key="name"
            dark
            color="amber"
            :rows-per-page-options="[0]"
          />

          <q-table
            class="my-sticky-header-column-table"
            style="width: 600px"
            title="Выплата в конце срока"
            :data="rows[2]"
            :columns="columns[2]"
            row-key="name"
            dark
            color="amber"
            :rows-per-page-options="[0]"
          />
        </div>

        <!-- <div class="finder">
            <q-btn
              v-if="send_status"
              @click="saved(
                sum,
                percent,
                mounth,
                date
                   )"
              style="margin: 0px 0px 0px 20px"
              icon-right="save"
              color='green'
              label='save'
              size="lg"
            >
            </q-btn>

            <q-btn
              v-else="send_status"
              style="margin: 0px 0px 0px 20px"
              icon-right="save"
              color='grey'
              label='save'
              size="lg"
            >
            </q-btn>
        </div> -->
      </div>
  </q-page>
</template>

<script>
import Chart from "components/Chart"
import axios from 'axios'
import {v4 as uuidv4} from 'uuid'
import anime from '../../anime'

export default {
  name: 'PageIndex',
  components: {Chart},
  data() {
    return {
      series: [
          {
            name: 'Аннуитентный платеж',
            type: 'line',
            smooth: true,
            itemStyle: {
              color: 'rgb(255, 118, 54)'
            },
            areaStyle: {
              opacity: 0.8,
              color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
                {
                  offset: 0,
                  color: 'rgb(255, 118, 54)'
                },
                {
                  offset: 1,
                  color: 'rgba(255, 255, 255, 0)'
                }
              ])
            },
            emphasis: {
              focus: 'series'
            },
            data: []
          },
          {
            name: 'Дифференциированный платеж',
            type: 'line',
            smooth: true,
            itemStyle: {
              color: 'rgb(91, 54, 255)'
            },
            emphasis: {
              focus: 'series'
            },
            areaStyle: {
              opacity: 0.8,
              color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
                {
                  offset: 0,
                  color: 'rgb(91, 54, 255)'
                },
                {
                  offset: 1,
                  color: 'rgba(255, 255, 255, 0)'
                }
              ])
            },
            data: []
          }
        ],

      xAxis: [
          {
            type: 'category',
            boundaryGap: false,
            data: []
          }
        ],

      // Прочий хлам

      rows: [[], [], []],
      columns: [],
      days: '',
      sum: '',
      percent: '',
      time: '',
      data: ['', '', '', ''],
      date: '2019/02/01',
      send_status: false,
      graf_status: false,
      list_status: false,
      massive: null,
      buffer: null,
      inp: true,
      credits: ['аннуитенный', 'дифференции- рованный'],
      height_var: null,
    }
  },
  methods: {

    masc(result) {
      this.massive = result
    },

    updateProxy () {
      this.time = this.date
    },

    save () {
      this.date = this.time
    },

    setStat() {
      this.send_status = false
    },

    async send(credit, percent, time, date_credit) {
      date_credit = new Date(date_credit).getTime()

      try {
        const response = await axios.post('api/graph', {
          credit: credit,
          percent: percent,
          time: time,
          credit_date: date_credit
        })

        this.massive = response.data.message
        this.buffer = response.data.buffer
      }

      catch(err) {
        return true
      }

      console.log(this.buffer)

      anime({
          targets: '.anime_load',
          translateX: -560,
          easing: 'easeInOutElastic(1, 1)',
          duration: 3000
        });

      anime({
          targets: '.tables_hidden',
          translateY: 164 + 48 * (this.massive.length+2),
          height: 0,
          duration: 2000 + this.massive.length*20,
          easing: 'easeInOutExpo'
        });

      await (this.graf_status = true)
      this.send_status = true
      
      this.data = [this.days, this.sum, this.percent, this.date]
      this.height_var = 164 + 48 * (this.massive.length+2)

      document.querySelector('.tables_hidden').style.setProperty('--height-var', this.height_var + 'px')

      setTimeout(this.tablesGen(this.massive[0].length), 100)
      this.grafData(this.massive[0].length)
      this.xAxisDate(this.massive[0].length)
      this.$refs.Chart.init()
    },

    saved(credit, percent, time, date_credit) {
      date_credit = new Date(date_credit).getTime()

      const response = axios.post('api/graph', {
        credit: credit,
        percent: percent,
        time: time,
        credit_date: date_credit,
        id: uuidv4()
      })
    },

    setDate() {
      let d = new Date();
      let month = (d.getMonth() + 1).toString()
      let day = d.getDate().toString()
      let year = d.getFullYear()
      if (month.length < 2) {
        month = '0' + month
      }
      if (day.length < 2) {
        day = '0' + day
      }
      this.date = [year, month, day].join('/')
    },
       
    rowsGen(time, graf) {
      this.rows[graf].splice(0, this.rows[graf].length)
      for( let i = 0; i < time; i++) {
        this.rows[graf].push({
          col2: this.massive[graf][i][2].slice(0, this.massive[graf][i][2].length - 14),
          col3: this.massive[graf][i][0], 
          col4: this.massive[graf][i][1],
          col5: +(this.massive[graf][i][0] + this.massive[graf][i][1]).toFixed(2)
        });
      }
      this.total(time - 1, graf)
    },

    endGen(time) {
      this.rows[2].splice(0, this.rows[2].length)
      for( let i = 0; i < time; i++) {
        this.rows[2].push({
          col2: this.massive[2][i][2].slice(0, this.massive[2][i][2].length - 14),
          col3: this.massive[2][i][0], 
          col4: this.massive[2][i][1],
          col5: 0
        });
      }
      this.rows[2].push({
          col2: this.massive[2][time][2].slice(0, this.massive[2][time][2].length - 14),
          col3: this.massive[2][time][0], 
          col4: this.massive[2][time][1],
          col5: this.totalSum(time, 2, 1) + this.massive[2][time][0]
        })
      this.total(time, 2)
    },

    totalSum(time, graf, col) {
      let sum = 0
      for(let i = 0; i <= time; i++) {
        sum += this.massive[graf][i][col]
      }
      return +sum.toFixed(2)
    },

    total(time, graf) {
      let cl2 = this.totalSum(time, graf, 0)
      let cl3 = this.totalSum(time, graf, 1)
      this.totalSum(time, graf, 1)
      this.rows[graf].push({
          col2: '== ИТОГО ==',
          col3: cl2,
          col4: cl3,
          col5: +(cl2 + cl3).toFixed(2)
        })
    },

    list() {
      if (!this.list_status) {
        this.list_status = true
        anime({
          targets: '.list_block',
          translateX: -300,
          easing: 'easeInOutExpo'
        });
      }

      else {
        this.list_status = false
        anime({
          targets: '.list_block',
          translateX: 0,
          easing: 'easeInOutExpo'
        });
      }
    },

    tablesGen(time){
      for( let i = 0; i < 2; i++) {
        this.rowsGen(time, i)
      }
      this.endGen(time - 1)
    },

    grafData(time) {
      for(let graf = 0; graf <= 1; graf++) {
        this.series[graf].data.splice(0, this.series[graf].data.length)
        for(let i = 0; i < time; i++) {
          this.series[graf].data.push(this.massive[graf][i][0] + this.massive[graf][i][1])
        }
      }
    },

    xAxisDate(time) {
      this.xAxis[0].data.splice(0, this.xAxis[0].data.length)
      for(let i = 0; i < time; i++) {
        this.xAxis[0].data.push(this.massive[0][i][2].slice(0, this.massive[0][i][2].length - 14))
      }
    },

    notification(message, type, condition=true) {
      if (condition) {
          this.$q.notify({
            type: type,
            message: message, 
            position: 'top-right'
          })
          this.inp = false
        }
    },

    testing() {
      this.inp = true

      this.notification('Не все данные были заполнены',
        'negative',
        this.days == '' || this.sum == '' || this.percent == '')

      this.notification('Срок должен быть целым числом',
        'warning',
        this.days - Math.ceil(this.days) != 0)

      this.notification('Срок на который берется кредит должен быть в пределах от 0 до 7320 дней',
        'warning',
        (0 >= this.days || this.days > 7320) && this.days != '')

      this.notification('Сумма кредита должна быть не менее 500 руб',
        'warning',
        (500 > this.sum) && this.sum != '')

      this.notification('Проценты должны быть в пределах от 0 до 50',
        'warning',
        (0 >= this.percent || this.percent > 50) && this.percent != '')

      if (this.inp) {
        this.ongoing()
      }
    },

    async ongoing () {

      document.querySelector('.graf_hidden').style.setProperty('--right-var', '-10px')
      document.querySelector('.graf_hidden').style.setProperty('--top-var', '-250px')

      const notif = this.$q.notify({
          type: 'ongoing',
          message: 'Загрузка'
        })

        // Проверка на наличие соединения

        if (await this.send(this.sum, this.percent, this.days, this.date)) {
            notif({
            type: 'negative',
            message: 'Ошибка сервера',
            timeout: 3000
          })
        }

        else {
          notif({
            type: 'positive',
            message: 'Запрос выполнен',
            timeout: 3000
          })
        }

      setTimeout(() => {
        anime({
          targets: '.graf',
          translateX: 1200,
          easing: 'easeInOutExpo',
          duration: 2000,
        });
      }, 500)

      setTimeout(() => {
        document.querySelector('.graf').style.setProperty('--z-index', '0')
      }, 3000)
    }
  },

  mounted() {
    anime({
          targets: '.header_image',
          translateY: 339,
          duration: 1700,
          easing: 'easeInOutExpo'
        });

    anime({
          targets: '.main_p',
          translateY: 839,
          duration: 1700,
          easing: 'easeInOutExpo'
        });
      
    anime({
          targets: '.list_button',
          translateY: 100,
          duration: 1700,
          easing: 'easeInOutExpo'
        });

    for(let i = 1; i <= 3; i++){
    this.columns.push([
          { name: 'col2', align: 'center', label: 'Дата', field: 'col2', sortable: true },
          { name: 'col3', align: 'center', label: 'Основной долг', field: 'col3', sortable: true },
          { name: 'col4', align: 'center', label: 'Начисленные проценты', field: 'col4', sortable: true },
          { name: 'col5', aling: 'center', label: 'Итого к оплате', field: 'col5', sortable: true }
        ])
    }

    this.setDate()
  }
}
</script>