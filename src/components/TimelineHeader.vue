<template>
    <thead class="bg-white text-center">
        <tr>
            <th></th>
            <th :colspan="month.n" v-for="month in monthsList" :key="'month-'+month.month">
                {{month.month}} {{month.start.getFullYear()}}
            </th>
        </tr>

        <tr>
            <th></th>
            <th :colspan="week.n" class="border-end-week" v-for="week in weeksList" :key="'week-'+week.week">
                Sem. {{week.week}}
            </th>
        </tr>

        <tr>
            <th class="bg-white">{{ listLabel }}</th>
            <th v-for="day in daysList" class="planning-cell" :key="'day-'+day" :class="{'border-end-week': day.getDay() === 0, 'text-primary': day.getSqlDate() == timeline.now.getSqlDate()}">
                <span class="d-block">{{dayDict[day.getDay()]}}</span>
                {{day.getDate()}}
            </th>
        </tr>
    </thead>
</template>

<script>
import '@/js/date.js';
import { mapState } from 'vuex';
import { listIntervalDays } from '../js/date';
import { listIntervalWeeks } from '../js/date';
import { listIntervalMonths } from '../js/date';

export default {
    props: {
        listLabel: String
    },

    data() {
        return {
            dayDict: ['Di', 'Lu', 'Ma', 'Me', 'Je', 'Ve', 'Sa'],
        }
    },

    computed: {
        ...mapState(['timeline']),
        
        /**
         * daysList()
         * Retourne la liste des dates (objet Date) start et end.
         * @returns {Array}
         */
        daysList() {
            return listIntervalDays(this.timeline.start, this.timeline.end);
        },

        /**
         * weeksList()
         * Retourne la liste des semaine entre start et end
         * @returns {array}
         */
        weeksList() {
            return listIntervalWeeks(this.timeline.start, this.timeline.end);
        },

        /**
         * monthsList()
         * Retourne la liste des mois entre start et end
         * @returns {array}
         */
        monthsList() {
            return listIntervalMonths(this.timeline.start, this.timeline.end);
        },
    },
}
</script>