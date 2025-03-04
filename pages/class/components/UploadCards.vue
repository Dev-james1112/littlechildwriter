<template>
  <v-tab-item class="pt-5">
    <v-row style="gap: 5px" class="ma-3">
      <v-select
        v-model="post.type"
        :items="['책', '공지사항', '파일', '링크', '글 제출 (학생)']"
        label="종류 선택"
        outlined
        class="mb-10"
      />
      <div>
        <v-combobox
          v-if="classInfo.uid === userInfo.uid"
          v-model="post.category"
          :items="Object.keys(classInfo.contents || {})"
          label="Search"
          outlined
          hide-selected
          clearable
          prepend-inner-icon="mdi-magnify"
        />
        <v-select
          v-else
          v-model="post.category"
          :items="Object.keys(classInfo.contents || {})"
          label="종류 선택"
          outlined
        />
      </div>
    </v-row>

    <v-card v-if="post.type === '책'" class="transparent">
      <v-card-title>책 업로드</v-card-title>
      <v-card-text>
        <v-text-field v-model="post.title" label="제목" />

        <div v-if="post.time">
          <h2>선택됨</h2>
          <br />
          <v-img :src="post.image" max-width="200" class="rounded-lg" />
        </div>
      </v-card-text>

      <v-card-actions class="ma-2 gap20">
        <v-dialog v-model="dialog" width="700">
          <template #activator="{ on, attrs }">
            <div class="text-center">
              <v-btn color="primary" v-bind="attrs" v-on="on">
                <v-icon left>mdi-bookshelf</v-icon> 책 선택
              </v-btn>
            </div>
          </template>

          <v-card>
            <v-row no-gutters>
              <v-card
                v-for="i in listev.filter(i => i.uid == userInfo.uid)"
                :key="i.title"
                class="elevation-0"
                @click="
                  post.time = i.time
                  post.image = i.image
                  dialog = false
                "
              >
                <v-img :src="i.image" class="rounded-lg ma-3" max-width="100" />
              </v-card>
            </v-row>
          </v-card>
        </v-dialog>

        <v-spacer />

        <v-btn
          :disabled="post.title === ''"
          color="primary"
          class="elevation-0"
          @click="Post"
        >
          게시
        </v-btn>
      </v-card-actions>
    </v-card>
    <v-card v-else-if="post.type === '링크'" class="transparent">
      <v-card-title>링크 업로드</v-card-title>
      <v-card-text>
        <v-text-field v-model="post.title" label="제목" />
        <v-text-field v-model="post.link" label="링크" />
      </v-card-text>
      <v-card-actions class="ma-2 gap20">
        <v-btn
          :disabled="post.title === ''"
          color="primary"
          class="elevation-0"
          @click="Post"
        >
          게시
        </v-btn>
      </v-card-actions>
    </v-card>
    <v-card v-else-if="post.type.startsWith('파일')" class="transparent">
      <v-radio-group v-model="post.type">
        <v-radio key="숙제" label="파일 (숙제로)" value="파일 (숙제로)" />
        <v-radio key="파일" label="파일" value="파일" />
        <v-radio key="사진" label="사진" value="파일 사진" />
        <v-radio key="비디오" label="비디오" value="파일 비디오" />
      </v-radio-group>

      <v-overlay :value="progress">
        <v-progress-circular indeterminate size="64" />
      </v-overlay>

      <v-file-input
        ref="file"
        v-model="post.file"
        color="deep-purple accent-4"
        counter
        label="File input"
        multiple
        placeholder="Select your files"
        prepend-icon="mdi-paperclip"
        outlined
        :show-size="1000"
        @change="UploadFile"
      >
        <template #selection="{ index, text }">
          <v-chip
            v-if="index < 2"
            color="deep-purple accent-4"
            dark
            label
            small
          >
            {{ text }}
          </v-chip>
        </template>
      </v-file-input>

      <v-btn text @click="Upload"> 파일 게시 </v-btn>
    </v-card>
    <v-card v-else-if="post.type === '글 제출 (학생)'" class="transparent">
      <v-card-title>숙제 업로드</v-card-title>
      <v-card-text>
        <v-text-field v-model="post.title" label="제목" />
        <v-textarea v-model="post.content" label="내용" />
      </v-card-text>
      <v-card-actions class="ma-2 gap20">
        <v-btn
          :disabled="post.title === ''"
          color="primary"
          class="elevation-0"
          @click="Post"
        >
          게시
        </v-btn>
      </v-card-actions>
    </v-card>
    <LazyCommentComponent
      v-else
      :link="`/class/${id}`"
      :dbr="`classes/${id}/contents/${post.category}`"
      :nocomments="true"
      :cb="() => updateTab(0)"
      :uid="userInfo.uid"
    />
  </v-tab-item>
</template>

<script setup lang="ts">
import { db, storage } from '@/plugins/firebase'
import { User } from '@/plugins/global'

const userInfo = User()
const route = useRoute()
const id = route.params.class
const classInfo = ref<any>({})
const listev = ref<any[]>([])
const post = ref<any>({
  isbn: '',
  title: '',
  image: '',
  pageCount: '',
  categories: [] as string[],
  rating: 5,
  content: '',
  uid: '',
  displayName: '',
  author: '',
  views: 0,
  time: Date.now(),
  isPublic: true,
  file: [] as File[],
  type: '책',
  category: '기타',
  book: true,
  link: ''
})
const dialog = ref<boolean>(false)
const progress = ref<boolean>(false)

const props = defineProps({
  updateTab: {
    type: Function,
    required: true
  }
})

onBeforeMount(() => {
  db.ref(`/classes/${id}`).on(
    'value',
    async s => (classInfo.value = await s.val())
  )

  db.ref('/contents/').on('child_added', s => {
    const { title, time, uid, displayName, image } = s.val()

    listev.value.push({
      title,
      time,
      uid,
      displayName,
      image
    })
  })
})

const Upload = () => {
  let storageRef: any

  for (let i = 0; i < post.value.file.length; i++) {
    storageRef = storage
      .ref(`${post.value.file[i].name}`)
      .put(post.value.file[i])
  }

  storageRef.on(
    'state_changed',
    (s: any) => (progress.value = s.bytesTransferred >= s.totalBytes),
    (e: Error) => Error(e.message),
    () =>
      storageRef.snapshot.ref
        .getDownloadURL()
        .then((url: any) => {
          const { type, file, category } = post.value
          const { uid, displayName } = userInfo.value

          db.ref(`classes/${id}/contents/${category}`).push({
            type,
            uid,
            displayName,
            url,
            file: file[0].name
          })
        })
        .then(() => {
          props.updateTab(0)
          post.value.title = ''
          post.value.time = 0
          post.value.file = []
          post.value.book = true
          post.value.type = '책'
          progress.value = false
        })
  )
}

const UploadFile = (f: File[]) => (post.value.file = f)

const Post = () => {
  const { title, time, category, type, content } = post.value
  const { uid, displayName } = userInfo.value

  if (type === '글 제출 (학생)') {
    db.ref(`classes/${id}/contents/${category}/${time}`).set({
      title,
      time,
      uid,
      displayName,
      type,
      content
    })
  } else {
    db.ref(`/classes/${id}/contents/${category}`).push({
      title,
      uid,
      time,
      displayName,
      type,
      content
    })
  }

  post.value = {
    isbn: '',
    title: '',
    image: '',
    pageCount: '',
    categories: [] as string[],
    rating: 5,
    content: '',
    uid: '',
    displayName: '',
    author: '',
    views: 0,
    time: Date.now(),
    isPublic: true,
    file: [] as File[],
    type: '포스트',
    category: '기타',
    book: true,
    link: ''
  }
  props.updateTab(0)
}
</script>

<style scoped>
.gap20 {
  gap: 20px;
}
</style>
