<template lang='jade' src='./timetable.jade'></template>
<style lang='stylus' src='./timetable.styl'></style>

<script>
import moment from 'moment';
import bells from 'bells';
import days from 'days';

export default {
  data() {
    return {
      days,
      bells,
      group: 1,
      error: '',
      timetable: [],
      timetables: [],
      actualLesson: 0,
      selectedTimetable: '',
      today: moment().day(),
    };
  },

  watch: {
    group(group) {
      localStorage.group = group;
    },

    selectedTimetable(name) {
      localStorage.selectedTimetable = name;

      this.fetchTimetable();
    },
  },

  filters: {
    removeGroup(name) {
      return name.replace(/-\d\/\d/, '');
    },
  },

  ready() {
    Object.assign(this, localStorage);
    this.detectLesson();

    fetch('http://localhost:8070/lista')
      .then(res => res.json())
      .then(list => {
        this.timetables = list;
        this.fetchTimetable();
      });
  },

  computed: {
    tb() {
      return this.timetable
        .map(column =>
          column.map(lesson => {
            const length = lesson.length;
            let res = null;

            if (!length) {
              res = lesson;
            } else if (length === 2) {
              res = lesson[this.group - 1];
            } else if (+/\d/.exec(lesson[0].subject)[0] === +this.group) {
              res = lesson[0];
            } else {
              res = { subject: '' };
            }

            return res;
          }));
    },
  },

  methods: {
    time(str) {
      return moment(`${moment().format('YYYY-MM-DD')} ${str}`);
    },

    detectLesson() {
      this.actualLesson = this.whichLesson();
    },

    tagLessons(timetable) {
      return timetable.map(day => day.map(this.lesson));
    },

    fetchTimetable() {
      this.error = '';

      fetch(`http://localhost:8070/plan/${this.timetables.indexOf(this.selectedTimetable) + 1}`)
        .then(res => res.json())
        .then(this.tagLessons)
        .then(this.rotate)
        .then(this.decorate)
        .then(tb => this.timetable = tb)
        .then(tb => {
          if (!tb.length) this.error = 'Sposób zapisu planu jest chwilowo niewspierany';
        })
        .catch(() => {
          if (this.selectedTimetable !== 'wybierz plan') {
            this.error = 'Sposób zapisu planu jest chwilowo niewspierany';
          }
        });
    },

    whichLesson() {
      let lesson = 0;

      bells.forEach((hour, i) => {
        if (moment().isBetween(this.time(hour[0]), this.time(hour[1]))) {
          lesson = i + 1;
        }
      });

      return lesson;
    },

    lesson(lesson) {
      const type = typeof lesson;

      if (type === 'object') {
        return lesson;
      }

      return { subject: '' };
    },

    rotate(timetable) {
      const rotated = [];

      for (let i = 5; i--;) {
        rotated[i] = [];

        timetable.forEach((t, j) => rotated[i][j] = t[i]);
      }

      return rotated;
    },
  },
};
</script>
