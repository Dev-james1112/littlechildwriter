<!-- eslint-disable vue/no-v-html -->
<template>
  <div>
    <iframe
      v-if="GBid"
      frameborder="0"
      scrolling="no"
      class="zmax frame"
      :src="`https://books.google.co.kr/books?id=${GBid}&lpg=PP1&pg=PP1&output=embed`"
      width="100%"
      height="100%"
    />
    <v-btn
      v-if="GBid"
      bottom
      right
      fixed
      color="primary"
      class="zmax"
      @click="GBid = ''"
    >
      닫기
    </v-btn>

    <v-card class="my-3 transparent">
      <div class="cardy">
        <div class="ma-auto">
          <v-bottom-sheet v-model="sheet">
            <template #activator="{ on, attrs }">
              <v-img
                :src="post.image"
                width="200"
                class="ml-5 my-5 rounded-lg pointer"
                v-bind="attrs"
                v-on="on"
              >
                <template #placeholder>
                  <v-row
                    class="fill-height ma-0"
                    align="center"
                    justify="center"
                  >
                    <v-progress-circular indeterminate color="grey lighten-5" />
                  </v-row>
                </template>
              </v-img>
            </template>
            <v-list>
              <v-subheader>외부 사이트</v-subheader>

              <v-list-item
                target="_blank"
                :href="`https://aladin.co.kr/shop/wproduct.aspx?isbn=${post.isbn}`"
              >
                <v-list-item-avatar>
                  <v-avatar size="32px">
                    <img
                      src="https://play-lh.googleusercontent.com/R83BmEu0bafVZ4lNC4dNnJ8Xxt9Cn5ZbS7m96SBaCgsxuTYaWINSgexcuSq8jhAvRkU"
                      alt="알라딘"
                    />
                  </v-avatar>
                </v-list-item-avatar>
                <v-list-item-title>알라딘</v-list-item-title>
              </v-list-item>

              <v-list-item
                target="_blank"
                :href="`https://www.yes24.com/product/search?query=${post.isbn}&domain=all`"
              >
                <v-list-item-avatar>
                  <v-avatar size="32px">
                    <img
                      src="https://play-lh.googleusercontent.com/ujk5bP2q6hYKSOnPrCRgJx4gW-oLdvFB3EfXQs6jdn11LOVRcPljdBqrgNBTxkFNRxY7"
                      alt="yes24"
                    />
                  </v-avatar>
                </v-list-item-avatar>
                <v-list-item-title>yes24</v-list-item-title>
              </v-list-item>

              <v-list-item
                target="_blank"
                :href="`https://www.amazon.com/s?k=${post.isbn}&i=stripbooks&linkCode=qs`"
              >
                <v-list-item-avatar>
                  <v-avatar size="32px">
                    <img
                      src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTt0-ykzLKIz1DSA5dDvSfrVy21kgN08CfsUw&usqp=CAU"
                      alt="yes24"
                    />
                  </v-avatar>
                </v-list-item-avatar>
                <v-list-item-title>아마존</v-list-item-title>
              </v-list-item>
            </v-list>
          </v-bottom-sheet>
        </div>

        <div class="ma-auto">
          <v-card-title class="h1 primary--text title">
            {{ post.title }}
            <span v-if="post.isbn" class="white--text subtitle-2 ml-1">
              ({{ (otherInfo.volumeInfo?.authors || []).join(', ') }})
            </span>
          </v-card-title>

          <v-card-subtitle>
            by
            <NLink :to="`/user/${post.uid}`">{{ post.displayName }}</NLink>
          </v-card-subtitle>

          <v-card-text>
            <v-rating
              dense
              readonly
              size="20"
              color="amber"
              background-color="white"
              :value="post.rating"
            />
            <div class="my-5" v-html="post.content" />
            <v-chip-group class="my-5" column>
              <v-chip label>
                <v-icon left> mdi-eye </v-icon>
                {{ formatter.format(post.views + 1) }}
              </v-chip>
              <v-chip label>
                <v-icon left>mdi-sort-clock-descending-outline</v-icon>
                {{ new Date(post.time).toLocaleDateString() }}
              </v-chip>
              <v-chip label>
                <v-icon left>mdi-book</v-icon>
                {{ post.pageCount }}
              </v-chip>
            </v-chip-group>
          </v-card-text>
        </div>
      </div>
    </v-card>

    <div class="text-center my-10">
      <div v-if="post.uid === userInfo.uid" class="mb-10"></div>

      <div v-if="post.isbn">
        <v-btn text @click="Iframe">
          <v-icon left> mdi-file-find </v-icon> 미리보기
        </v-btn>
        <v-dialog
          v-if="post.categories"
          transition="dialog-bottom-transition"
          width="700"
        >
          <template #activator="{ on, attrs }">
            <v-btn text v-bind="attrs" v-on="on">
              <v-icon left> mdi-shape </v-icon> 카테고리
            </v-btn>
          </template>

          <v-card>
            <v-card-text>
              <br />
              <v-chip
                v-for="tag in post.categories"
                :key="tag"
                ripple
                outlined
                label
                :to="`/list?search=${tag}`"
                class="ma-2 d-block"
              >
                #{{ tag }}
              </v-chip>
            </v-card-text>
          </v-card>
        </v-dialog>
        <v-dialog
          v-if="post.isbn"
          transition="dialog-bottom-transition"
          width="700"
        >
          <template #activator="{ on, attrs }">
            <v-btn text v-bind="attrs" v-on="on">
              <v-icon left> mdi-book-information-variant </v-icon> 정보
            </v-btn>
          </template>

          <v-card>
            <v-card-title>
              {{ post.title }}
            </v-card-title>
            <v-card-subtitle>
              {{ (otherInfo.volumeInfo?.authors || []).join(', ') }}
            </v-card-subtitle>

            <v-card-text>
              <v-simple-table>
                <template #default>
                  <thead>
                    <tr>
                      <th class="text-left">Categories</th>
                      <th class="text-left">Information</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr>
                      <td>ISBN 13</td>
                      <td>{{ post.isbn }}</td>
                    </tr>
                    <tr>
                      <td>ISBN 10</td>
                      <td>
                        {{
                          otherInfo.volumeInfo.industryIdentifiers[1].identifier
                        }}
                      </td>
                    </tr>
                    <tr>
                      <td>출판된 날짜</td>
                      <td>{{ otherInfo.volumeInfo?.publishedDate }}</td>
                    </tr>
                    <tr>
                      <td>출판사</td>
                      <td>{{ otherInfo.volumeInfo.publisher }}</td>
                    </tr>
                    <tr v-if="otherInfo.GBid">
                      <td>Google Books ID</td>
                      <td>{{ otherInfo.GBid }}</td>
                    </tr>
                    <tr>
                      <td>평균 별점 (구글)</td>
                      <td>{{ otherInfo.volumeInfo.averageRating }}</td>
                    </tr>
                  </tbody>
                </template>
              </v-simple-table>
            </v-card-text>
          </v-card>
        </v-dialog>
      </div>
    </div>

    <LazyCommentComponent
      :link="`/book/content/${time}`"
      :dbr="`contents/${time}/comments`"
      :uid="post.uid"
    />

    <br /><br /><br /><br /><br /><br />

    <div class="text-center">
      <v-progress-circular v-if="loading" indeterminate color="primary" large />
    </div>

    <template v-if="post.categories && !loading">
      <h1>이런 책 어때?</h1>

      <v-row>
        <v-card
          v-for="(item, i) in suggested"
          :key="i"
          class="mx-auto my-5 transparent"
          :href="item.infoLink"
          :width="$vuetify.breakpoint.width < 1264 ? 150 : 210"
        >
          <v-img :src="item.thumbnail" class="rounded-lg" />
        </v-card>
      </v-row>
    </template>

    <br /><br /><br />

    <v-speed-dial
      v-if="userInfo.uid === post.uid"
      v-model="fab"
      bottom
      right
      open-on-hover
      direction="top"
      transition="slide-y-reverse-transition"
      style="position: fixed"
    >
      <template #activator>
        <v-btn v-model="fab" color="blue darken-2" dark fab>
          <v-icon v-if="fab"> mdi-cog-off </v-icon>
          <v-icon v-else> mdi-cog </v-icon>
        </v-btn>
      </template>
      <v-btn fab dark small color="green" :to="`/book/edit/${time}`">
        <v-icon>mdi-pencil</v-icon>
      </v-btn>
      <v-btn fab dark small color="red" @click="Del">
        <v-icon>mdi-delete</v-icon>
      </v-btn>
    </v-speed-dial>
  </div>
</template>

<script setup lang="ts">
import { db } from '@/plugins/firebase'
import { formatter, Libris, User } from '@/plugins/global'

const userInfo = User()
const router = useRouter()
const route = useRoute()
const time = route.params.content
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
  views: 0,
  time: Date.now()
})
const otherInfo = ref<any>({
  volumeInfo: {
    authors: [],
    publisher: '',
    publishedDate: '',
    averageRating: 0
  },
  GBid: ''
})
const suggested = ref<any>([])
const GBid = ref<string>('')
const loading = ref<boolean>(true)
const sheet = ref<boolean>(false)
const fab = ref<boolean>(false)

const Content = async () => {
  const data = await db
    .ref(`/contents/${time}`)
    .once('value')
    .then(r => r.val())

  data !== null && Object.keys(data).length !== 1 && (post.value = data)

  if (data.isbn) {
    await fetch(
      `https://www.googleapis.com/books/v1/volumes?q=isbn:${data.isbn}`
    )
      .then(res => res.json())
      .then(res => (otherInfo.value = res.items[0]))
  }
}

const View = () => {
  db.ref(`contents/${time}/views`).transaction(view => view + 1)
  Libris(post.value.uid, 0.1)
  Libris(userInfo.value.uid, 0.1)

  if (userInfo.value.uid === post.value.uid) {
    db.ref(`contents/${time}/views`).transaction(view => view - 1)
    post.value.views--
  }
}

const Suggestion = async () => {
  loading.value = true

  if (!post.value.categories) {
    loading.value = false
    return
  }

  const cat: string[] = [...post.value.categories]
  cat.forEach((tag, i) => (cat[i] = encodeURIComponent(`'${tag}'`)))

  let done = false
  let n = 5
  let overflow = 0

  while (!done) {
    overflow++

    if (overflow > 10) {
      done = true
      break
    }

    await fetch(
      `https://www.googleapis.com/books/v1/volumes?q=subject:${cat.join(
        ','
      )}&maxResults=${n}`
    )
      .then(res => res.json())
      .then(data => {
        const length = data.items.length

        if (length > 4) {
          done = true
        } else {
          cat.shift()
          n -= length
        }

        for (let i = 0; i < length; i++) {
          const book = data.items[i]

          if (
            book.volumeInfo.imageLinks.thumbnail &&
            book.volumeInfo.title !== post.value.title
          ) {
            const {
              imageLinks: { thumbnail },
              infoLink
            } = book.volumeInfo

            suggested.value.push({
              thumbnail,
              infoLink
            })

            if (i === 4) {
              break
            }
          }
        }
      })
      .catch(() => cat.shift())
  }

  loading.value = false
}

onBeforeMount(async () => {
  await Content()
  View()
  Suggestion()
})

const Iframe = async () => {
  let fetched = ''

  await fetch(
    `https://www.googleapis.com/books/v1/volumes?q=isbn:${post.value.isbn}`
  )
    .then(res => res.json())
    .then(res => (fetched = res.items[0].id))

  GBid.value = fetched
}

const Del = () => {
  db.ref(`contents/${time}`).remove()
  Libris(userInfo.value.uid, -(parseInt(post.value.pageCount) / 100))
  router.push('/list')
}

useHead({
  title: '컨텐츠 - LCW'
})
</script>

<style scoped>
.frame {
  position: absolute;
  border: 0px;
  top: 0px;
  left: 0px;
}

.cardy {
  display: flex;
  padding: 10px;
}

.pointer {
  cursor: pointer;
}

@media screen and (max-width: 605px) {
  .cardy {
    display: block;
  }
}
</style>
