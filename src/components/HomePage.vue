<script setup lang="ts">
import { ref, onMounted } from "vue";
import axios from "axios";

interface Post {
  id: number;
  title: { rendered: string };
  author: number;
  featured_media: number;
  date?: string;
  link: string;
}

interface Author {
  id: number;
  name: string;
}

interface Category {
  id: number;
  name: string;
  parent: number;
  children?: Category[];
}

const post = ref<{
  post: Post | null;
  postAuthor: string;
}>({
  post: null,
  postAuthor: "",
});

const heroBackgroundStyle = ref({
  background: "white",
  display: "block",
  href: "",
});

const heroTitleStyle = ref({
  position: "absolute",
  background:
    "linear-gradient(0deg, rgba(0,0,0,1) 5%, rgba(255,255,255,0) 100%)",   
});

const parseDate = (date: any) => {
  const newDate = new Date(date);
  const readableDate = ref(
    newDate.toLocaleString("en-US", {
      year: "numeric",
      month: "long",
      day: "numeric",
    })
  );

  return readableDate;
};

onMounted(async () => {
  // Posts
  try {
    const tagID = 69; // Replace 1 with the ID of the category you want to display
    const response = await axios.get(
      `https://theluzonian.press/wp-json/wp/v2/posts?tags=${tagID}`
    );
    if (response.data.length > 0) {
      post.value.post = response.data[0];
      if (post.value.post?.featured_media) {
        const response = await axios.get(
          "https://theluzonian.press/wp-json/wp/v2/media/" +
            post.value.post.featured_media
        );
        heroBackgroundStyle.value.background =
          "url(" + response.data.guid.rendered + ")";
      } else {
        heroBackgroundStyle.value.display = "none";
        heroTitleStyle.value.position = "static";
        heroTitleStyle.value.background = "#800000";
      }
    } else {
      console.log("No posts found");
    }
  } catch (error) {
    console.error("Error fetching posts:", error);
  }

  // Authors
  try {
    const response = await axios.get(
      "https://theluzonian.press/wp-json/wp/v2/users?per_page=100"
    );
    post.value.postAuthor = response.data.find(
      (x: Author) => x.id === post.value.post?.author
    ).name;
  } catch (error) {
    console.error("Error fetching posts:", error);
  }
});

const decodedText = (dataText: string) => {
  const parser = new DOMParser();
  const dom = parser.parseFromString(
    `<!doctype html><body>${dataText}`,
    "text/html"
  );
  return dom.body.textContent;
};
</script>

<template>
  <section v-if="post.post">
    <a :href="post.post.link" style="text-decoration: none; color: inherit;">

    <div style="position: relative">
      <div class="hero-background" :style="heroBackgroundStyle"></div>
      <div class="hero-title text-white" :style="heroTitleStyle">
          <img
            class="logo"
            src="https://theluzonian.press/wp-content/uploads/2024/01/3Asset-16DAEL-LOGO-.png"
            alt="dael_logo"
          />
          <div class="pt-8" style="max-width: 120rem">

            <h3 class="hero-title-heading">
              {{ decodedText(post.post.title.rendered) }}
            </h3>
           
            <p
              class="text-body-1 pt-2"
              style="
                font-family: sans-serif;
                margin: auto;
                width: 80vw;
                padding-bottom: 40px;
              "
            >
              <b>{{ decodedText(post.postAuthor) }}</b> |
              {{ parseDate(post.post.date).value }}
            </p>
          </div>
        
      </div>
    </div>
  </a>
  </section>
</template>


<style>

.hero-background {
  min-height: 60vh;
  min-width: none;
  background-size: cover !important;
  z-index: -1;
  background-position: 50% 40% !important;
  background-repeat: no-repeat !important;
}

.hero-title {
  font-family: sans-serif;
  font-weight: bolder !important;
  color: "#fff";
  font-size: 3rem;
  z-index: 3;
  bottom: 0rem;
  width: 100%;
  display: flex;
  justify-content: space-between;
  flex-direction: column;
  height: 100%;
  min-height: 25rem;
  line-height: 110%;
}

.hero-title-heading {
  font-family: sans-serif;
  color: white;
  font-weight: bolder;
  font-size: 50px;
  margin: auto;
  width: 80vw;
  max-width: 1200px;
}

.logo {
  max-width: 18rem;
  padding: 40px;

}

/* mobile phone */
@media only screen and (max-width: 768px) {
  
  .hero-title-heading {
    font-family: sans-serif;
    color: white;
    font-weight: bolder;
    font-size: 28px;
    line-height: 2rem;
    margin: auto;
    width: 80vw;
    max-width: 1200px;
  }

  .logo {
    max-width: 14rem;
    padding: 40px;
  }
}

/* tablet */
@media only screen and (min-width: 768px) and (max-width: 1024px) {
  .hero-title-heading {
    font-family: sans-serif;
    color: white;
    font-weight: bolder;
    font-size: 40px;
    line-height: 2.5rem;
    margin: auto;
    width: 80vw;
    max-width: 1200px;
  }

  .logo {
    max-width: 16rem; 
    padding: 40px; 
  }
}


</style>
