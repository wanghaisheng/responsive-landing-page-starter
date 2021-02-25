<template>
  <div
    class="flex p-px align-top rounded-lg shadow-lg bg-gradient-to-b"
    :class="gradient"
  >
    <component
      :is="internal ? 'nuxt-link' : 'a'"
      :to="internal ? localePath(link) : ''"
      :href="!internal ? link : ''"
      :target="!internal ? '_blank' : ''"
      class="flex flex-col overflow-hidden rounded-lg"
    >
      <section class="flex flex-col items-start flex-1 p-6 bg-white">
        <h2 class="mb-4 text-lg">
          <svg
            xmlns="http://www.w3.org/2000/svg"
            fill="none"
            viewBox="0 0 24 24"
            stroke="currentColor"
            class="inline-block mr-1 stroke-current stroke-2 icon-size"
          >
            <path
              strokeLinecap="round"
              strokeLinejoin="round"
              strokeWidth="{2}"
              :d="svgPath"
            />
          </svg>
          {{ title }}
        </h2>
        <p>
          {{ description }}
        </p>
      </section>
    </component>
  </div>
</template>

<script>
export default {
  props: {
    title: {
      type: String,
      required: true,
    },
    description: {
      type: String,
      required: true,
    },
    link: {
      type: String,
      required: true,
    },
    svgPath: {
      type: String,
      required: true,
    },
    gradient: {
      type: String,
      required: true,
    },
  },

  computed: {
    internal() {
      return !this.link.startsWith('http')
    },
  },
}
</script>
