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
      timetable: [],
      actualLesson: 0,
      today: moment().day(),
    };
  },

  watch: {
    group(group) {
      localStorage.group = group;
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

    fetch('http://localhost:8070')
      .then(res => res.json())
      .then(this.tagLessons)
      .then(this.rotate)
      .then(this.decorate)
      .then(tb => this.timetable = tb);
  },

  computed: {
    diag() {
      return JSON.stringify(this.timetable);
    },

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
      return moment(`11-05-1997 ${str}`);
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

    detectLesson() {
      this.actualLesson = this.whichLesson();
    },

    lesson(lesson) {
      const type = typeof lesson;

      if (type === 'object') {
        return lesson;
      }

      return { subject: '' };
    },

    tagLessons(timetable) {
      return timetable.map(day => day.map(this.lesson));
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
