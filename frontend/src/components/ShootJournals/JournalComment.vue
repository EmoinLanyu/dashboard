<!--
Copyright (c) 2020 by SAP SE or an SAP affiliate company. All rights reserved. This file is licensed under the Apache Software License, v. 2 except as noted otherwise in the LICENSE file

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

     http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
-->

<template>
  <v-row dense>
    <v-col style="max-width: 60px;">
      <v-avatar size="40px">
        <img :src="avatarUrl" :title="login"/>
      </v-avatar>
    </v-col>
    <v-col class="fluid" >
      <div class="comment">
        <div class="comment-header">
          <span style="font-weight: 700">{{login}}</span> commented <a :href="htmlUrl" target="_blank"><time-string :dateTime="createdAt" :pointInTime="-1"></time-string></a>
        </div>
        <div class="comment-body" v-html="compiledMarkdown"></div>
      </div>
    </v-col>
  </v-row>

</template>

<script>
import get from 'lodash/get'
import { compileMarkdown } from '@/utils'
import TimeString from '@/components/TimeString'

export default {
  components: {
    TimeString
  },
  props: {
    comment: {
      type: Object,
      required: true
    }
  },
  computed: {
    compiledMarkdown () {
      return compileMarkdown(get(this.comment, 'data.body', ''))
    },
    login () {
      return get(this.comment, 'data.user.login')
    },
    createdAt () {
      return get(this.comment, 'metadata.created_at')
    },
    avatarUrl () {
      return get(this.comment, 'data.user.avatar_url')
    },
    htmlUrl () {
      return get(this.comment, 'data.html_url')
    }
  }
}
</script>

<style lang="scss" scoped>

  .comment {
    border-radius: 2px;
    border: 0.5px;
    border-color: #00acc1;
    border-style: solid;
    margin-right: 16px;
  }
  .comment-header {
    border-top-left-radius: 2px;
    border-top-right-radius: 2px;
    border: 0.5px;
    padding: 5px;
    border-bottom: 0.5px solid #00acc1;
    background-color: #00acc1;
    color: white;
    font-weight: 400;
  }
  .comment-body {
    padding: 5px;

    /* not needed for chrome, but kept for firefox */
    word-wrap: break-word;
    /* does not work with firefox */
    word-break: break-word;
  }

  ::v-deep .flex {
    width: 200px;
  }

  a {
    text-decoration: none;
    color: white;
  }

  a:hover{
    text-decoration: underline;
  }
</style>
