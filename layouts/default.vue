<template>
  <v-app>
    <v-navigation-drawer
      v-model="bookmark"
      fixed
      app
      temporary
      :color="$vuetify.theme.dark ? '#23262E' : '#f5f5f5'"
    >
      <v-card-title> <v-icon left>mdi-bookmark</v-icon> 북마크 </v-card-title>
      <v-list v-if="items.length > 0">
        <v-list-item
          v-for="(item, i) in items"
          :key="i"
          :to="`/book/content/${item.time}`"
        >
          <v-list-item-content>
            <v-list-item-title> {{ item.title }} </v-list-item-title>
          </v-list-item-content>
          <v-list-item-action>
            <v-spacer />
            <v-menu offset-y>
              <template #activator="{ on, attrs }">
                <v-btn icon v-bind="attrs" cols="1" v-on="on" @click.prevent>
                  <v-icon>mdi-dots-vertical</v-icon>
                </v-btn>
              </template>
              <v-list>
                <v-list-item @click="deleteBookmark(item.time, i)">
                  <v-list-item-content>
                    <v-list-item-title>
                      <v-icon left> mdi-trash-can </v-icon>
                      삭제
                    </v-list-item-title>
                  </v-list-item-content>
                </v-list-item>
              </v-list>
            </v-menu>
          </v-list-item-action>
        </v-list-item>
      </v-list>
      <v-card v-else class="transparent">
        <v-card-text> 북마크가 없습니다. </v-card-text>
      </v-card>

      <br />
      <v-divider />
      <br />

      <v-card class="transparent">
        <v-card-actions>
          <v-btn color="primary" text @click="$vuetify.theme.dark = false">
            <v-icon left> mdi-lightbulb-on </v-icon>
            Light
          </v-btn>
          <v-btn color="primary" text @click="$vuetify.theme.dark = true">
            <v-icon left> mdi-lightbulb-on-outline </v-icon>
            Dark
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-navigation-drawer>

    <v-app-bar
      fixed
      app
      outlined
      :collapse="$route.path.startsWith('/class')"
      :color="$vuetify.theme.dark ? '#23262E' : '#f5f5f5'"
    >
      <v-app-bar-nav-icon v-if="userInfo.uid" @click="bookmark = !bookmark" />

      <NLink to="/">
        <v-avatar size="30">
          <img src="/icon.png" alt="Logo" />
        </v-avatar>
      </NLink>

      <v-spacer />

      <div v-if="!$route.path.startsWith('/class')">
        <v-slide-x-transition>
          <v-btn
            v-if="$route.path !== '/book/post' && userInfo.uid"
            icon
            to="/book/post"
          >
            <v-icon>mdi-plus</v-icon>
          </v-btn>
        </v-slide-x-transition>

        <v-dialog
          v-if="userInfo.uid"
          v-model="notifOverlay"
          transition="dialog-bottom-transition"
          width="700"
          scrollable
        >
          <template #activator="{ on, attrs }">
            <v-btn icon class="amber--text" v-bind="attrs" v-on="on">
              <v-badge
                overlap
                left
                transition
                :content="notif.length"
                :value="notif.length"
              >
                <v-icon>
                  {{
                    notif.length > 0
                      ? 'mdi-bell-ring-outline'
                      : 'mdi-bell-outline'
                  }}
                </v-icon>
              </v-badge>
            </v-btn>
          </template>

          <v-card>
            <v-list v-if="notif.length > 0">
              <v-list-item
                v-for="(d, i) in notif"
                :key="i"
                @click="load(d.link)"
              >
                <v-list-item-avatar>
                  <v-img :src="d.photoURL" />
                </v-list-item-avatar>

                <v-list-item-content>
                  <v-list-item-title>{{ d.title }}</v-list-item-title>
                  <v-list-item-subtitle>{{ d.time }}</v-list-item-subtitle>
                </v-list-item-content>
              </v-list-item>
            </v-list>
            <div v-else>
              <v-card-text class="text-center grey--text">
                알림이 없습니다.
              </v-card-text>
            </div>

            <v-card-actions>
              <v-spacer />
              <v-btn text @click="clearEverything">
                비우기 <v-icon right> mdi-notification-clear-all </v-icon>
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>

        <v-menu v-if="userInfo.uid" right>
          <template #activator="{ on }">
            <v-btn icon v-on="on">
              <v-avatar size="35">
                <v-img alt="User Avatar" :src="userInfo.photoURL" />
              </v-avatar>
            </v-btn>
          </template>

          <v-card class="d-flex pa-3 text-center rounded-0">
            <v-avatar size="35" class="ma-auto">
              <v-img alt="User Avatar" :src="userInfo.photoURL" />
            </v-avatar>

            <div>
              <v-card-title>{{ userInfo.displayName }}</v-card-title>
              <v-card-subtitle>{{ userInfo.email }}</v-card-subtitle>
            </div>
          </v-card>

          <v-list>
            <v-list-item to="/account/account">
              <v-list-item-title>
                <v-icon left>mdi-cog-outline</v-icon> 설정
              </v-list-item-title>
            </v-list-item>
            <v-list-item :to="`/user/${userInfo.uid}`">
              <v-list-item-title>
                <v-icon left>mdi-account-circle</v-icon> 프로필
              </v-list-item-title>
            </v-list-item>

            <v-divider />

            <v-list-item @click="logout">
              <v-list-item-title>
                <v-icon left>mdi-logout</v-icon> 로그아웃
              </v-list-item-title>
            </v-list-item>
          </v-list>
        </v-menu>
        <v-btn v-else to="/account/login" icon>
          <v-icon>mdi-account-circle</v-icon>
        </v-btn>
      </div>
    </v-app-bar>

    <v-main>
      <v-container>
        <br v-if="!$route.path.startsWith('/class')" />
        <Nuxt />
        <br />
      </v-container>
    </v-main>
  </v-app>
</template>

<script setup lang="ts">
import { auth, db } from '@/plugins/firebase'
import { User } from '@/plugins/global'

const router = useRouter()
const userInfo = User()
const notif = ref<any>([])
const items = ref<any>([])
const notifOverlay = ref<boolean>(false)
const bookmark = ref<boolean>(false)

onMounted(() => {
  auth.onAuthStateChanged(u => {
    if (!u) {
      return
    }

    db.ref(`/users/${u.uid}/bookmarks`).on('child_added', async s =>
      items.value.push(await s.val())
    )

    db.ref(`/users/${u.uid}/notification`).on('child_added', async s =>
      notif.value.push(await s.val())
    )

    db.ref('.info/connected').on('value', s => {
      if (s.val()) {
        db.ref(`/users/${userInfo.value.uid}/status`)
          .onDisconnect()
          .set('offline')
        db.ref(`/users/${userInfo.value.uid}/status`).set('online')
      }
    })
  })
})

const clearEverything = () => {
  db.ref(`/users/${userInfo.value.uid}/notification`).remove()
  notif.value = []
}

const load = (link: string) => {
  notifOverlay.value = false
  router.push(link)
}

const deleteBookmark = (time: string, i: number) => {
  db.ref(`/users/${userInfo.value.uid}/bookmarks/${time}`).remove()
  db.ref(`/contents/${time}/bookmarks/${userInfo.value.uid}`).remove()
  items.value.splice(i, 1)
}

const logout = () => {
  auth.signOut()
  router.push('/account/login')
  userInfo.value = {
    displayName: '',
    email: '',
    photoURL: '',
    uid: ''
  }
}
</script>
