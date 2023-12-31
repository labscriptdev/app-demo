<template>
  <v-defaults-provider
    :defaults="{
      VTextField: { variant: 'outlined' },
      VTextarea: { variant: 'outlined' },
      VFileInput: { variant: 'outlined' },
      VSelect: { variant: 'outlined' },
      VCombobox: { variant: 'outlined' },
    }"
  >

    <!-- Loading -->
    <template v-if="!app.ready">
      <div class="d-flex align-center justify-center" style="height:100vh;">
        <div>
          <div class="text-disabled">Loading</div>
          <v-progress-linear indeterminate />
        </div>
      </div>
    </template>

    <!-- Auth -->
    <template v-if="app.ready && (!app.user || (app.user && !app.user.app_user_group))">
      <div class="d-flex align-center justify-center bg-grey-lighten-3" style="height:100vh;">
        <div style="width:400px; max-width:90vw;">
          <v-card>
            <v-tabs v-model="auth.tab">
              <v-tab value="login">Login</v-tab>
              <v-tab value="register">Register</v-tab>
              <v-tab value="password">Password</v-tab>
            </v-tabs>

            <v-alert type="error" rounded="0" v-if="app.user && !app.user.app_user_group">
              Sorry {{ app.user.name }}. You are not an admin.
            </v-alert>

            <v-window v-model="auth.tab">
              <v-window-item value="login">
                <v-card-text>
                  <app-auth-login />
                  <br>
                  <app-auth-switch />
                </v-card-text>
              </v-window-item>
              <v-window-item value="register">
                <v-card-text>
                  <app-auth-register />
                </v-card-text>
              </v-window-item>
              <v-window-item value="password">
                <v-card-text>
                  <app-auth-password />
                </v-card-text>
              </v-window-item>
            </v-window>
          </v-card>
        </div>
      </div>
    </template>


    <!-- Logged -->
    <template v-if="app.ready && app.user && app.user.app_user_group">
      <v-layout>
        <v-app-bar @click="drawer.main=true">
          <v-btn icon="mdi-menu" flat class="d-lg-none"></v-btn>
          <v-toolbar-title>
            <span class="font-weight-bold me-2">
              {{ app.settings['app.name'] }}
            </span>
            <span class="text-subtitle-1 text-muted">
              Welcome {{ app.user.name }}
            </span>
          </v-toolbar-title>
          <v-spacer></v-spacer>

          <div>
            <v-btn stacked>
              <v-badge :model-value="false" color="primary">
                <v-icon>mdi-chat</v-icon>
              </v-badge>
            </v-btn>
          </div>

          <!-- Notifications -->
          <div>
            <v-menu :close-on-content-click="false">
              <template #activator="{ props }">
                <v-btn v-bind="props" stacked>
                  <v-badge
                    :model-value="notification.success ? notification.success.data.length : false"
                    :content="notification.success ? notification.success.data.length : null"
                    color="primary"
                  >
                    <v-icon>mdi-bell</v-icon>
                  </v-badge>
                </v-btn>
              </template>

              <v-card width="300" class="mt-3">
                <v-progress-linear indeterminate v-if="notification.loading" />
                <v-window v-model="notifications.tab">
                  <v-window-item value="list">
                    <v-list>
                      <v-list-subheader>
                        Notifications
                      </v-list-subheader>
                      <v-divider></v-divider>
                      <div
                        v-if="notification.success && notification.success.data.length==0"
                        class="text-center text-disabled py-3"
                      >
                        <div class="mb-2">Nothing to see</div>
                        <v-icon size="40">mdi-emoticon-happy-outline</v-icon>
                      </div>
                      <v-list-item
                        v-for="n in notification.success.data"
                        @click="notifications.view(n)"
                      >
                        <v-list-item-text>
                          {{ n.name }}
                        </v-list-item-text>
                      </v-list-item>
                    </v-list>
                  </v-window-item>
                  <v-window-item value="view">
                    <v-card-text class="border-b font-weight-bold">
                      {{ notifications.data.name }}
                    </v-card-text>
                    <v-card-text>
                      <div
                        v-html="notifications.data.text"
                        style="white-space: break-spaces;"
                      ></div>
                    </v-card-text>
                    <v-card-actions class="border-t">
                      <v-spacer />
                      <v-btn @click="notifications.setTab('list')">Back</v-btn>
                    </v-card-actions>
                  </v-window-item>
                </v-window>
              </v-card>
            </v-menu>
          </div>
        </v-app-bar>
        
        <v-navigation-drawer
          v-model="drawer.main"
          v-bind="{
            width: 250,
            border: 0,
            class: 'border-e',
          }"
        >
          <div class="d-flex flex-column" style="height: calc(100vh - 64px);">
            <div class="flex-grow-1" style="overflow:auto;">
              <slot name="sidebar">
                <app-nav variant="plain" :items="[
                  {to:'/admin', name:'Dashboard'},
                  {to:'/admin', name:'Users', icon:'mdi-account', children: [
                    {to:'/admin/app_user', name:'Users list'},
                    {to:'/admin/app_user_group', name:'Groups list'},
                  ]},
                  {to:'/', name:'Settings', icon:'mdi-cog', children: [
                    {to:'/admin/settings', name:'Configurações'},
                    {to:'/admin/app_mail_template', name:'E-mail templates'},
                    {to:'/admin/app_file', name:'Uploads'},
                    {to:'/admin/app_place', name:'Places'},
                  ]},
                ]" />
              </slot>
            </div>
            <app-nav
              variant="plain"
              density="compact"
              :items="[
                {name:'Switch account', onClick() { drawer.account = true; }},
                {name:'Sair', onClick() { app.logout(); }},
              ]"
            />
          </div>
        </v-navigation-drawer>
  
        <v-main>
          <v-container class="pt-8 pt-md-10" style="max-width:1200px;">
            <template v-if="hasPermission">
              <slot></slot>
            </template>
            <template v-else>
              <v-alert type="error">
                Sorry, you have no permission to see this screen.
              </v-alert>
            </template>
          </v-container>
        </v-main>
      </v-layout>
    </template>


    <!-- Switch account -->
    <v-dialog v-model="drawer.account">
      <v-card style="width:400px; max-width:90vw; margin:0 auto;">
        <v-card-title>Switch account</v-card-title>
        <v-divider />
        <v-card-text>
          <app-auth-login />
          <br>
          <app-auth-switch @switch="drawer.account=false" />
        </v-card-text>
        <v-divider />
        <v-card-actions>
          <v-spacer />
          <v-btn @click="drawer.account=false">Close</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-defaults-provider>
</template>


<script setup>
  import { ref, computed, defineProps } from 'vue';
  import { breakpointsVuetify, useBreakpoints } from '@vueuse/core';
  import axios from 'axios';
  
  import useApp from '@/composables/useApp';
  const app = useApp();
  
  import useWebsocket from '@/composables/useWebsocket';
  const websocket = useWebsocket({
    events: [
      ['app_user_notification@created', (data) => {
        notification.value.submit();
      }],
    ],
  });

  const notification = useAxios({
    method: 'get',
    url: 'api://app_user_notification',
    params: {
      user_id: 'me',
      status: "unread",
    },
    autoSubmit: true,
  });

  const notifications = ref({
    tab: 'list',
    data: false,
    async view(notif) {
      this.tab = 'view';
      this.data = notif;
      notif.status = 'read';
      try {
        const { data } = await axios.post(`api://app_user_notification`, notif);
        this.data = data;
        notification.value.submit();
      } catch(err) {}
    },
    setTab(tab) {
      this.tab = tab;
    },
  });

  const route = useRoute();

  const hasPermission = computed(() => {
    if (!app.ready) return false;
    const permissionKey = `view:${route.name}`;
    if (typeof app.permissions[permissionKey] != 'undefined') {
      return app.user.app_user_group.permissions.includes(permissionKey);
    }
    return true;
  });

  const breakpoints = useBreakpoints(breakpointsVuetify);

  const props = defineProps({
    containerFluid: {
      type: Boolean,
      default: false,
    },
  });

  const drawer = ref({
    main: breakpoints.md,
    account: false,
  });

  const auth = ref({
    tab: 'login',
  });
</script>

<style>
  @import url('https://fonts.googleapis.com/css2?family=Montserrat:ital,wght@0,100;0,200;0,300;0,400;0,500;0,600;0,700;0,800;0,900;1,100;1,200;1,300;1,400;1,500;1,600;1,700;1,800;1,900&display=swap');
  html, body { font-family: 'Montserrat', sans-serif; }
</style>
