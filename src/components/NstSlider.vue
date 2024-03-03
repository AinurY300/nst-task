<template>
  <vue-slider
    class="nst-slider"
    v-model="sliderValue"
    :max="sliderData.length - 1"
    :marks="true"
    :tooltip-placement="['top', 'bottom']"
    tooltip='always'
    >
    <template v-slot:tooltip="{ value: id }: { value: SliderData['id'] }">
      <div>{{ sliderData[id].monthFullName }}</div>
      <div>{{ sliderData[id].year }}</div>
    </template>

    <template v-slot:label="{ value: id }: { value: SliderData['id'] }">  
      <div :class="['vue-slider-mark-label', sliderData[id].monthName === 'янв' && showMonths ? 'year-label' : 'month-label']">
        {{ sliderData[id].label }}
      </div>
    </template>
    
  </vue-slider>
</template>

<script setup lang="ts">
import 'vue-slider-component/theme/default.css'
import VueSlider from 'vue-slider-component'
import { ref, watch, onMounted, withDefaults, onUpdated } from 'vue'


type SliderData = {
  id: number,
  year: number,
  monthNum: Month['number'],
  monthName: Month['name'],
  monthFullName: Month['fullName'],
  label: SliderData['monthName'] | SliderData['year'] | ''
}

type Month = {
  number: 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12,
  name: 'янв'|'фев'|'мар'|'апр'|'май'|'июн'|'июл'|'авг'|'сен'|'окт'|'ноя'|'дек',
  fullName: 'Январь'|'Февраль'|'Март'|'Апрель'|'Май'|'Июнь'|'Июль'|'Август'|'Сентябрь'|'Октябрь'|'Ноябрь'|'Декабрь'
}

const months: Month[] = [
  { number:  1, name: 'янв', fullName: 'Январь'  },
  { number:  2, name: 'фев', fullName: 'Февраль' },
  { number:  3, name: 'мар', fullName: 'Март'    },
  { number:  4, name: 'апр', fullName: 'Апрель'  },
  { number:  5, name: 'май', fullName: 'Май'     },
  { number:  6, name: 'июн', fullName: 'Июнь'    },
  { number:  7, name: 'июл', fullName: 'Июль'    },
  { number:  8, name: 'авг', fullName: 'Август'  },
  { number:  9, name: 'сен', fullName: 'Сентябрь'},
  { number: 10, name: 'окт', fullName: 'Октябрь' },
  { number: 11, name: 'ноя', fullName: 'Ноябрь'  },
  { number: 12, name: 'дек', fullName: 'Декабрь' }
]


const props = withDefaults(defineProps<{
  minYear?: number,
  maxYear?: number,
  minMonth?: Month['number'],
  maxMonth?: Month['number'],
  selectedStartYear?: number,
  selectedEndYear?: number,
  selectedStartMonth?: Month['number'],
  selectedEndMonth?: Month['number'],
  showMonths?: boolean
  modelValue?: {
    start: SliderData,
    end: SliderData,
  }
}>(), {
  minYear: 2014,
  maxYear: 2021,
  minMonth: 1,
  maxMonth: 1,
  showMonths: true,
})

const emits = defineEmits<{
  (e: 'update:modelValue', value: typeof props.modelValue): void
}>()


const minYear = ref(props.minYear)
const maxYear = ref(props.maxYear)
const minMonth = ref(props.minMonth)
const maxMonth = ref(props.maxMonth)
const sliderData = ref<SliderData[]>([])


// На основе переменных выше заполняем массив sliderData.
const generateData = function() {
  const data: typeof sliderData['value'] = []
  
  for (let year = minYear.value; year <= maxYear.value; year++) {
    const startMonthNum = year === minYear.value ? minMonth.value : 1
    const endMonthNum   = year === maxYear.value ? maxMonth.value : 12

    for (let monthNum = startMonthNum; monthNum <= endMonthNum; monthNum++) {
      const month = months.find(month => month.number === monthNum) as Month

      data.push({
        id: data.length,
        year: year,
        monthNum: month.number,
        monthName: month.name,
        monthFullName: month.fullName,
        label: '' // Потом поменяю
      })
    }
  }
  return data
}
sliderData.value = generateData() // Тут понятно


// Устанавливаем значения по умолчанию на основе пропсов.
const startDate = sliderData.value.find(i => {
  if (props.selectedStartYear && props.selectedStartMonth) return i.year === props.selectedStartYear && i.monthNum === props.selectedStartMonth
  else if (props.selectedStartYear && !props.selectedStartMonth) return i.year === props.selectedStartYear
  else if (!props.selectedStartYear && props.selectedStartMonth) return i.monthNum === props.selectedStartMonth
}) as SliderData

const endDate = sliderData.value.find(i => {
  if (props.selectedEndYear && props.selectedEndMonth) return i.year === props.selectedEndYear && i.monthNum === props.selectedEndMonth
  else if (props.selectedEndYear && !props.selectedEndMonth) return i.year === props.selectedEndYear
  else if (!props.selectedEndYear && props.selectedEndMonth) return i.monthNum === props.selectedEndMonth && i.year >= startDate?.year
}) as SliderData

const sliderValue = ref([
  startDate?.id || 0,
  endDate?.id || sliderData.value.length - 1
])




// Если меняется значение "showMonths", то по новой заполняем массив "sliderData"
watch(() => props.showMonths, (showMonths) => {
  const oldMinYear = sliderData.value[sliderValue.value[0]].year
  const oldMaxYear = sliderData.value[sliderValue.value[1]].year
  const oldMinMonth = sliderData.value[sliderValue.value[0]].monthNum
  const oldMaxMonth = sliderData.value[sliderValue.value[1]].monthNum

  if (showMonths) {
    minYear.value  = oldMinYear
    maxYear.value  = oldMaxYear
    minMonth.value = oldMinMonth
    maxMonth.value = oldMaxMonth

    sliderData.value = generateData()
    sliderValue.value = [0, sliderData.value.length - 1]

  } else {
    minYear.value = props.minYear
    maxYear.value = props.maxYear
    minMonth.value = props.minMonth
    maxMonth.value = props.maxMonth

    sliderData.value = generateData()
    const startValue = sliderData.value.findIndex(i => i.monthNum === oldMinMonth && i.year === oldMinYear)
    const endValue = sliderData.value.findIndex(i => i.monthNum === oldMaxMonth && i.year === oldMaxYear)
    sliderValue.value = [startValue, endValue]
  }
})




watch(sliderValue, (value) => {
  emits("update:modelValue", {
    start: sliderData.value.find(i => i.id === value[0]) as SliderData,
    end: sliderData.value.find(i => i.id === value[1]) as SliderData
  })
}, { immediate: true })



/* 
  Для нормального отображения подсказок при ограниченной ширине слайдера
  я решил не отображать некоторые месяцы в зависимости от их количества
*/
const labelAdaptation = function() {
  const rail = document.querySelector('.vue-slider-marks')
  
  if (rail) {
    sliderData.value.forEach(item => {
      item.label = props.showMonths ? (item.monthNum === 1 ? item.year : item.monthName) : (item.monthNum === 1 ? item.year : '')
    })
    
    const getMarksWidth = () => sliderData.value.reduce((acc, i) => i.label != '' ? acc += 30 : acc = acc, 0) // 30 - ширина подсказки

    // Удаление 1: Оставляем половину.
    if (rail.clientWidth < getMarksWidth()) {
      sliderData.value.forEach(item => [2,4,6,8,10,12].includes(item.monthNum) ? item.label = '' : '')
    } 
    // Удаление 2: Оставляем май и сентябрь.
    if (rail.clientWidth < getMarksWidth()) {
      sliderData.value.forEach(item => [2,3,4,6,7,8,10,11,12].includes(item.monthNum) ? item.label = '' : '')
    }
    // Удаление 2: Оставляем только июль.
    if (rail.clientWidth < getMarksWidth()) {
      sliderData.value.forEach(item => {
        item.label = Number(item.label) ? item.label : (item.monthNum === 7 ? item.monthName : '')
      })
    }
    // Удаление 4: Да кому нужны эти ваши месяцы - не отображаем месяцы.
    if (rail.clientWidth < getMarksWidth()) {
      sliderData.value.forEach(item => {
        item.label = Number(item.label) ? item.label : ''
      })
    }
  }
}

onMounted(() => window.addEventListener('resize', labelAdaptation))
onUpdated(() => labelAdaptation())
</script>

<style>
.vue-slider {
  height: 8px !important;
  padding: 2px !important;
  background: var(--surface-variant);
  border-radius: 10px;
}

.vue-slider-rail {
  background: none;
}

.vue-slider-process {
  background: var(--primary);
}

.vue-slider-mark-step {
  display: none;
}

.vue-slider-mark-label {
  font-weight: 600;
  opacity: .4 !important;
  width: 24px !important;
  font-size: .75em;
}

.vue-slider-mark-label.display-none {
  display: none !important;
}

@media (max-width: 760px) {
  .vue-slider-mark-label {
    font-size: .7em;
  }
}

.vue-slider-mark-label.year-label {
  font-weight: 700;
  opacity: 1 !important;
}

.vue-slider-dot-handle {
  width: 18px;
  height: 18px;
  cursor: pointer;
  border-radius: 50%;
  position: relative;
  background: var(--primary);
  transform: translate(-2px, -3px);
  box-shadow: 0 0 4px -1px var(--shadow);
}

.vue-slider-dot-handle::after {
  top: 50%;
  left: 50%;
  width: 9px;
  height: 9px;
  content: '';
  display: block;
  position: absolute;
  background: #fff;
  border-radius: 50%;
  transform: translate(-50%, -50%);
}

.vue-slider-dot-tooltip {
  color: var(--blue-text);
  margin-top: -8px;
  padding: 8px 10px;
  font-size: .8em;
  font-weight: 700;
  text-align: center;
  background: #fff;
  border-radius: 10px;
  box-shadow: 0 0 10px -5px #002747;
  position: relative;
}

.vue-slider-dot-tooltip::after {
  left: 50%;
  z-index: 1;
  content: "";
  bottom: -18px;
  position: absolute;
  border-width: 10px;
  border-style: solid;
  transform: translateX(-50%);
  border-color: #fff transparent transparent transparent;
}

.vue-slider-dot-tooltip-bottom {
  margin-bottom: -8px;
}

.vue-slider-dot-tooltip-bottom::after {
  left: 50%;
  content: "";
  position: absolute;
  border-width: 10px;
  border-style: solid;
  bottom: calc(100% - 1px);
  transform: translateX(-50%);
  border-color: transparent transparent #fff transparent;
}
</style>

<!-- 
⣿⣿⣿⠟⠛⠛⠻⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⡟⢋⣩⣉⢻
⣿⣿⣿⠀⣿⣶⣕⣈⠹⠿⠿⠿⠿⠟⠛⣛⢋⣰⠣⣿⣿⠀⣿
⣿⣿⣿⡀⣿⣿⣿⣧⢻⣿⣶⣷⣿⣿⣿⣿⣿⣿⠿⠶⡝⠀⣿
⣿⣿⣿⣷⠘⣿⣿⣿⢏⣿⣿⣋⣀⣈⣻⣿⣿⣷⣤⣤⣿⡐⢿
⣿⣿⣿⣿⣆⢩⣝⣫⣾⣿⣿⣿⣿⡟⠿⠿⠦⠀⠸⠿⣻⣿⡄⢻
⣿⣿⣿⣿⣿⡄⢻⣿⣿⣿⣿⣿⣿⣿⣿⣶⣶⣾⣿⣿⣿⣿⠇⣼
⣿⣿⣿⣿⣿⣿⡄⢿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⡟⣰
⣿⣿⣿⣿⣿⣿⠇⣼⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⢀⣿
⣿⣿⣿⣿⣿⠏⢰⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⢸⣿
⣿⣿⣿⣿⠟⣰⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⠀⣿
⣿⣿⣿⠋⣴⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⡄⣿
⣿⣿⠋⣼⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⡇⢸
⣿⠏⣼⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⡯⢸
 -->