/* eslint-disable */


<script setup lang="ts">
import { ref, onMounted } from 'vue';
import axios from 'axios';

interface Post {
  id: number;
  title: { rendered: string };
  author: number;
  featured_media: number; // Add this line
  date?: string; // Add this if you have dates in your posts
  // Add other post properties as needed
}


interface Author {
  id: number;
  name: string;
  // Add other author properties as needed
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
  postAuthor: ""
});

const heroBackgroundStyle = ref({
  background: 'url("https://theluzonian.press/wp-content/uploads/2024/01/cropped-Transparent-Logo.png")',
  display: 'block',
});

const heroTitleStyle = ref({
  position: 'absolute',
  background: 'linear-gradient(0deg, rgba(0,0,0,1) 25%, rgba(255,255,255,0) 100%)'
});



const parseDate = (date:any) => {
  const newDate = new Date(date);
  const readableDate = ref(newDate.toLocaleString('en-US', {
    year: 'numeric',
    month: 'long',
    day: 'numeric',
  }));

  return readableDate
}

onMounted(async () => {
  // Posts
  try {
    const response = await axios.get('https://theluzonian.press/wp-json/wp/v2/posts');
    if (response.data.length > 0) {
      post.value.post = response.data[0];
      if(post.value.post?.featured_media){
        const response = await axios.get('https://theluzonian.press/wp-json/wp/v2/media/'+ post.value.post.featured_media);
        heroBackgroundStyle.value.background = "url("+response.data.guid.rendered+")";
      }else{
        heroBackgroundStyle.value.display="none"
        heroTitleStyle.value.position="static"
        heroTitleStyle.value.background="#800000"
      }
    } else {
      console.log('No posts found');
    }
  } catch (error) {
    console.error('Error fetching posts:', error);
  }

  // Authors
  try {
    const response = await axios.get('https://theluzonian.press/wp-json/wp/v2/users');
    post.value.postAuthor = response.data.find((x: Author) => x.id === post.value.post?.author).name
  } catch (error) {
    console.error('Error fetching posts:', error);
  }



});

const decodedText = (dataText:string) => {
  const parser = new DOMParser();
  const dom = parser.parseFromString(
    `<!doctype html><body>${dataText}`,
    'text/html'
  );
  return dom.body.textContent;
};

</script>


<template>
  <section v-if="post.post" >
    <div style="position: relative;">
      <div class="hero-background" :style="heroBackgroundStyle"></div>
      <div class="hero-title text-white pa-10" :style="heroTitleStyle">
        <img src="https://theluzonian.press/wp-content/uploads/2024/01/3Asset-16DAEL-LOGO-.png" alt="dael_logo" style="max-width: 13rem;">
        <div class="pt-8" style="max-width: 120rem;">
          <h3 style="font-family: 'Geologica-Regular',Helvetica,Arial,Lucida,sans-serif; color: white; font-weight: bolder; font-size: 50px;">{{ decodedText(post.post.title.rendered) }}</h3>
          <p class="text-body-1 pt-2" style="font-family: 'Geologica', sans-serif;">
            <b>{{ decodedText(post.postAuthor) }}</b> |
            {{ parseDate(post.post.date).value }}

          </p>
        </div>

      </div>
      <div class="text-overlay"></div>
      <img class="hero-logo" src=""/>

    </div>

  </section>
</template>

<style>
  @import url('https://fonts.googleapis.com/css2?family=Anton&family=Geologica:wght@100;200;300;400;500;600;700;800;900&display=swap');

  .hero-background{
    min-height: 35rem;
    min-width: none;
    background-size: cover!important;
    z-index: -1;
    background-position: center!important;
    background-repeat: no-repeat!important;
  }

  .hero-title{
    font-family: 'Geologica-Regular',Helvetica,Arial,Lucida,sans-serif;
    font-weight: bolder!important;
    color:"#fff";
    font-size:3rem;
    z-index:3;
    bottom: 0rem;
    text-transform: uppercase;
    width: 100vw;
    display: flex;
    justify-content: space-between;
    flex-direction: column;
    height: 100%;
    min-height: 25rem;
    line-height: 110%;

  }

</style>
