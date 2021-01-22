<template>
  <div>
    <HeroImage
      :src="require('~/assets/images/illustrations/Spotlight-banner.png')"
      alt="Spotlight Posts"
    />
    <main class="max-w-screen-xl px-4 mx-auto sm:px-6 lg:px-8">
      <Breadcrumbs />
      <section class="index-section">
        <div class="grid grid-cols-1 gap-6 md:grid-cols-2">
          <article
            class="flex flex-col overflow-hidden rounded-lg shadow-lg bg-white p-6"
          >
            <h2 class="text-lg font-bold mb-2">
              <svg
                xmlns="http://www.w3.org/2000/svg"
                fill="none"
                viewBox="0 0 24 24"
                stroke="currentColor"
                class="inline-block mr-1 stroke-current stroke-2 icon-size text-blue"
              >
                <path
                  strokeLinecap="round"
                  strokeLinejoin="round"
                  strokeWidth="{2}"
                  d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"
                />
              </svg>
              What We're Looking For
            </h2>
            <p class="mb-2">
              We're interested in both technical tutorials and general pieces on
              programming. Successful submissions will be:
            </p>
            <p class="text-sm mb-2">
              <strong>Original.</strong> Literally, we want original work
              (nothing that has been published elsewhere). And more
              figuratively, we want to see original thought and interesting
              points of view we can't find anywhere else.
            </p>
            <p class="text-sm mb-2">
              <strong>Well-written.</strong> Our standards are high, and we want
              everything published on our blog to meet those standards. That
              said, you don't have to be a professional writer to submit! If the
              post is original, helpful and interesting, our editorial team is
              happy to work with you to get it publication ready.
            </p>

            <h2 class="text-lg font-bold mb-2 mt-4">
              <svg
                xmlns="http://www.w3.org/2000/svg"
                fill="none"
                viewBox="0 0 24 24"
                stroke="currentColor"
                class="inline-block mr-1 stroke-current stroke-2 icon-size text-blue"
              >
                <path
                  strokeLinecap="round"
                  strokeLinejoin="round"
                  strokeWidth="{2}"
                  d="M10 20l4-16m4 4l4 4-4 4M6 16l-4-4 4-4"
                />
              </svg>
              Does my post have to be about Vonage products?
            </h2>
            <p class="mb-2">Your posts should involve Vonage products.</p>
            <p class="text-sm mb-2">
              It can be any of them: SMS, Voice, Video, Verify, Conversation
              APIs, or one of the other tools of our communication suite.
            </p>

            <h2 class="text-lg font-bold mb-2 mt-4">
              <svg
                xmlns="http://www.w3.org/2000/svg"
                fill="none"
                viewBox="0 0 24 24"
                stroke="currentColor"
                class="inline-block mr-1 stroke-current stroke-2 icon-size text-blue"
              >
                <path
                  strokeLinecap="round"
                  strokeLinejoin="round"
                  strokeWidth="{2}"
                  d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.94 3.31.826 2.37 2.37a1.724 1.724 0 001.065 2.572c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 00-1.066 2.573c.94 1.543-.826 3.31-2.37 2.37a1.724 1.724 0 00-2.572 1.065c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066c-1.543.94-3.31-.826-2.37-2.37a1.724 1.724 0 00-1.065-2.572c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 001.066-2.573c-.94-1.543.826-3.31 2.37-2.37.996.608 2.296.07 2.572-1.065z"
                />
                <path
                  strokeLinecap="round"
                  strokeLinejoin="round"
                  strokeWidth="{2}"
                  d="M15 12a3 3 0 11-6 0 3 3 0 016 0z"
                />
              </svg>
              The Process
            </h2>
            <p class="mb-2">
              When we receive your submission, our editorial team will evaluate
              it to see if it's a good fit for the Vonage blog.
            </p>
            <p class="text-sm mb-2">
              Submission is not a guarantee that your work will be published; we
              reserve the right to decline submissions that aren't right for our
              platform. But we really want to help you get published so if you
              take the time to submit, we will review your work and provide
              feedback. Our goal is simple: we want to see what you're doing
              with Vonages's products, and we want you to share your story with
              the community.
            </p>
            <p class="text-sm mb-2">
              We'll be in touch to let you know if we want to publish your post
              and tell you what the next steps are.
            </p>

            <h2 class="text-lg font-bold mb-2 mt-4">
              <svg
                xmlns="http://www.w3.org/2000/svg"
                fill="none"
                viewBox="0 0 24 24"
                stroke="currentColor"
                class="inline-block mr-1 stroke-current stroke-2 icon-size text-blue"
              >
                <path
                  strokeLinecap="round"
                  strokeLinejoin="round"
                  strokeWidth="{2}"
                  d="M7 16V4m0 0L3 8m4-4l4 4m6 0v12m0 0l4-4m-4 4l-4-4"
                />
              </svg>
              Who Owns Your Work?
            </h2>
            <p class="mb-2">
              You do! You grant Vonage an unlimited license to publish and
              promote your post, but you retain full ownership.
            </p>
            <p class="text-sm mb-2">
              This means you are free to re-publish elsewhere (on your own
              Medium page, for example). All we require is that you add a
              <code>rel=canonical</code> tag to your post wherever possible,
              naming the original post at Vonage the canonical URL.
            </p>
          </article>
          <article
            class="flex flex-col overflow-hidden rounded-lg shadow-lg bg-white p-6"
          >
            <h2 class="text-lg font-bold mb-2">
              <svg
                xmlns="http://www.w3.org/2000/svg"
                fill="none"
                viewBox="0 0 24 24"
                stroke="currentColor"
                class="inline-block mr-1 stroke-current stroke-2 icon-size text-blue"
              >
                <path
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  stroke-width="{2}"
                  d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z"
                />
              </svg>
              What's in it for You?
            </h2>
            <p class="mb-2">
              We pay $500 USD per post. Your tutorial will be published on our
              blog, and may be shared in our newsletter or showcased as part of
              our "Tutorial Tuesday" weekly feature.
            </p>
            <p class="text-sm mb-2">
              We make every effort to amplify the reach of each and every blog
              post on our site, and yours will be no exception; you'll have the
              Vonage marketing and social media teams behind you getting the
              word out about your work. Your success is our success.
            </p>
            <p class="mb-2">
              And Now Introducing:
              <a
                class="text-purple"
                href="https://www.nexmo.com/blog/2019/09/04/nexmo-launches-spotlight-for-donations-program-dr"
                >Spotlight for Donations</a
              >
            </p>
            <p class="text-sm mb-2">
              Authors can now choose to pay it forward and donate their $500 to
              a charity making a difference in the technology industry.
            </p>
            <h2 class="text-lg font-bold mt-4">
              <svg
                xmlns="http://www.w3.org/2000/svg"
                fill="none"
                viewBox="0 0 24 24"
                stroke="currentColor"
                class="inline-block mr-1 stroke-current stroke-2 icon-size text-blue"
              >
                <path
                  strokeLinecap="round"
                  strokeLinejoin="round"
                  strokeWidth="{2}"
                  d="M3 8l7.89 5.26a2 2 0 002.22 0L21 8M5 19h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v10a2 2 0 002 2z"
                />
              </svg>
              Submit Your Idea
            </h2>
            <form
              class="space-y-6"
              name="spotlight"
              method="POST"
              data-netlify="true"
              netlify-honeypot="some-info"
            >
              <div class="hidden">
                <input type="hidden" name="form-name" value="spotlight" />
                <label
                  >Don’t fill this out if you are human:
                  <input name="some-info"
                /></label>
              </div>
              <div>
                <label
                  for="name"
                  class="block text-sm font-medium text-gray-700"
                >
                  Your name
                </label>
                <div class="mt-1">
                  <input
                    id="name"
                    name="name"
                    type="name"
                    autocomplete="name"
                    required
                    class="appearance-none block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm placeholder-gray-400 focus:outline-none focus:ring-indigo-500 focus:border-purple sm:text-sm"
                  />
                </div>
              </div>
              <div>
                <label
                  for="email_address"
                  class="block text-sm font-medium text-gray-700"
                >
                  Your email address
                </label>
                <div class="mt-1">
                  <input
                    id="email_address"
                    name="email_address"
                    type="email_address"
                    autocomplete="email_address"
                    required
                    class="appearance-none block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm placeholder-gray-400 focus:outline-none focus:ring-indigo-500 focus:border-purple sm:text-sm"
                  />
                </div>
              </div>
              <div>
                <label
                  for="background"
                  class="block text-sm font-medium text-gray-700"
                >
                  Please write a little about your background and why you want
                  to write for Vonage
                </label>
                <div class="mt-1">
                  <textarea
                    id="background"
                    name="background"
                    rows="3"
                    class="appearance-none block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm placeholder-gray-400 focus:outline-none focus:ring-indigo-500 focus:border-purple sm:text-sm"
                  ></textarea>
                </div>
              </div>
              <div>
                <label
                  for="outline"
                  class="block text-sm font-medium text-gray-700"
                >
                  The outline of your blog post idea
                </label>
                <div class="mt-1">
                  <textarea
                    id="outline"
                    name="outline"
                    rows="3"
                    class="appearance-none block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm placeholder-gray-400 focus:outline-none focus:ring-indigo-500 focus:border-purple sm:text-sm"
                  ></textarea>
                </div>
              </div>
              <div>
                <label
                  for="previous_content"
                  class="block text-sm font-medium text-gray-700"
                >
                  Please provide link to something you have written previously
                </label>
                <div class="mt-1">
                  <input
                    id="previous_content"
                    name="previous_content"
                    type="previous_content"
                    autocomplete="previous_content"
                    required
                    class="appearance-none block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm placeholder-gray-400 focus:outline-none focus:ring-indigo-500 focus:border-purple sm:text-sm"
                  />
                </div>
              </div>
              <p class="mt-2 text-sm text-gray-500">
                By submitting your ideas to Vonage Developer Spotlight you
                accept our
                <a
                  href="https://www.vonage.com/legal/privacy-policy/"
                  target="_blank"
                  class="text-purple"
                  >privacy policy</a
                >, which sets out how we use your data and the rights you have
                in respect of your data.
              </p>
              <div>
                <button
                  type="submit"
                  name="commit"
                  class="w-full flex justify-center py-2 px-4 border border-transparent rounded-md shadow-sm text-sm text-white bg-black border-black hover:bg-white hover:text-black active:bg-purple-dark active:text-white active:border-purple-dark focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-purple-dark font-bold"
                >
                  Submit my idea
                </button>
              </div>
            </form>
          </article>
        </div>
      </section>
      <section class="index-section">
        <h2>
          <nuxt-link :to="localePath(`/blog/spotlight`)">
            Spotlight Posts
          </nuxt-link>
        </h2>
        <div class="grid grid-cols-1 gap-6 md:grid-cols-2 lg:grid-cols-3">
          <Card v-for="(post, i) in posts" :key="i" :post="post" />
        </div>
      </section>
    </main>
  </div>
</template>

<script>
import config from '~/modules/config'

export default {
  async asyncData({ $content, app, error }) {
    try {
      const posts = await $content(`blog/${app.i18n.locale}`)
        .where({
          $and: [{ spotlight: { $eq: true } }, { published: { $ne: false } }],
        })
        .sortBy('published_at', 'desc')
        .fetch()

      if (posts.length === 0) {
        error({ statusCode: 404, message: 'Page not found' })
      }

      return { posts }
    } catch (e) {
      return error(e)
    }
  },

  head() {
    return {
      title: 'Developer Spotlight Programme',
      meta: [
        {
          hid: 'description',
          name: 'description',
          content:
            'Tutorials and Blog Posts from our Spotlight Authors. Earn up to $500 USD and get featured on our platform.',
        },
        {
          hid: 'twitter:title',
          name: 'twitter:title',
          // Team Members & Authors » Developer Content from Vonage ♥
          content: `Developer Spotlight Programme${config.baseSplitter}${config.baseTitle}`,
        },
        {
          hid: 'twitter:description',
          name: 'twitter:description',
          content:
            'Tutorials and Blog Posts from our Spotlight Authors. Earn up to $500 USD and get featured on our platform.',
        },
        {
          hid: 'og:title',
          property: 'og:title',
          // {author name} » Developer Content from Vonage ♥
          content: `Developer Spotlight Programme${config.baseSplitter}${config.baseTitle}`,
        },
        {
          hid: 'og:description',
          property: 'og:description',
          content:
            'Tutorials and Blog Posts from our Spotlight Authors. Earn up to $500 USD and get featured on our platform.',
        },
      ],
    }
  },
}
</script>
