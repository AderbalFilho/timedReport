<!-- eslint-disable vue/no-v-html -->
<template>
  <div
    class="mb-4 col-12 col-sm-6 col-md-4 col-lg-3"
    data-hj-whitelist
  >
    <div
      :id="HTMLCardId"
      class="card text-center h-100"
      :class="{ selected: isSelected }"
      @contextmenu="$emit('contextmenu', $event)"
    >
      <div class="close-wrapper">
        <button
          type="button"
          class="close close-card"
          aria-label="Close"
          @click="$emit('card-closed')"
        >
          <span aria-hidden="true">&times;</span>
        </button>
      </div>

      <div
        class="card-header eta"
        @click="cardClickedOr('edit-eta', 'eta')"
        v-html="etaString"
      ></div>

      <div
        class="card-body d-flex flex-column"
        @click.self="$emit('card-clicked')"
      >
        <div
          class="card-title h5"
          data-bound-property="title"
          @click="cardClickedOr('edit-title', 'card-title')"
        >
          {{ title }}
        </div>

        <div
          class="card-subtitle mb-2 text-muted"
          data-bound-property="project"
          @click="cardClickedOr('edit-project', 'card-subtitle')"
        >
          {{ project }}
        </div>

        <p
          class="card-text"
          style="flex-grow:inherit"
          data-bound-property="description"
          @click="cardClickedOr('edit-description', 'card-text')"
        >
          {{ description }}
        </p>

        <div
          class="progress my-2"
          style="height: 2px;"
        >
          <div
            class="progress-bar bg-secondary"
            role="progressbar"
            :style="{ width: progress+'%', 'background-color': progressColorCss }"
            :aria-valuenow="progress"
            aria-valuemin="0"
            aria-valuemax="100"
          ></div>
        </div>

        <div
          class="card-status"
          @click="$emit('card-clicked')"
        >
          {{ taskStateString }}
        </div>
      </div>

      <div
        class="card-footer timer"
        @click="cardClickedOr('edit-time', 'timer')"
        v-html="timeString"
      ></div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    title: {
      type: String,
      required: true,
    },
    project: {
      type: String,
      required: true,
    },
    description: {
      type: String,
      required: true,
    },

    progressColor: {
      type: String,
      required: true,
    },
    progress: {
      type: Number,
      required: true,
    },
    taskStateString: {
      type: String,
      required: true,
    },

    time: {
      type: Number,
      required: true,
    },
    eta: {
      type: Number,
      required: true,
    },

    isSelected: Boolean,
    isEditing: Boolean,
  },
  data() {
    return {
      HTMLCardId: this.$vnode.key,
      delay: 200,
      clicks: 0,
      timer: null,
    };
  },
  computed: {
    timeString() {
      return new Date(this.time + this.timeOffset()).toTimeString().match(/\d\d:\d\d:\d\d/)[0];
    },
    etaString() {
      return new Date(this.eta + this.timeOffset()).toTimeString().match(/\d\d:\d\d:\d\d/)[0];
    },
    progressColorCss() {
      if (this.progressColor === undefined) {
        return '';
      }

      return `${this.progressColor}!important`;
    },
  },
  mounted() {
    const card = document.querySelector(`#${this.HTMLCardId}`);

    const elements = [
      card.querySelector('.card-title'),
      card.querySelector('.card-subtitle'),
      card.querySelector('.card-text'),
    ].filter((identity) => Boolean(identity));

    elements.forEach((element) => {
      element.addEventListener('paste', this.pastePureText);
    });
  },
  methods: {
    timeOffset() {
      return new Date(0).getTimezoneOffset() * 60000;
    },
    cardClickedOr(eventName, field) {
      this.clicks += 1;

      if (this.clicks === 1) {
        const self = this;

        this.timer = setTimeout(() => {
          self.$emit('card-clicked');
          self.clicks = 0;
        }, this.delay);
      } else {
        clearTimeout(this.timer);

        const elementToEdit = $(`#${this.HTMLCardId} .${field}`)[0];

        this.$emit(eventName, elementToEdit);
        this.clicks = 0;
      }
    },
    pastePureText(e) {
      e.preventDefault();
      const text = e.clipboardData.getData('text/plain');
      const temp = document.createElement('div');
      temp.innerHTML = text;
      document.execCommand('insertHTML', false, temp.textContent);
    },
  },
};
</script>
