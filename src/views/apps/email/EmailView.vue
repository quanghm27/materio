<template>
  <div class="h-full">
    <!-- 1st Row -->
    <div class="d-flex align-center px-sm-6 px-4 py-2">
      <!-- Back Button -->
      <v-btn
        icon
        small
        class="me-2"
        @click="$emit('close-email-view')"
      >
        <v-icon size="28">
          {{ icons.mdiChevronLeft }}
        </v-icon>
      </v-btn>

      <!-- Subject -->
      <h1 class="font-weight-medium text-base me-2 text-truncate">
        {{ emailViewData.subject }}
      </h1>

      <!-- Labels -->
      <v-badge
        v-for="label in emailViewData.labels"
        :key="label"
        :color="resolveLabelColor(label)"
        inline
        :content="label"
        class="email-label-chip text-capitalize v-badge-light-bg"
        :class="`${resolveLabelColor(label)}--text`"
      >
      </v-badge>
      <v-spacer></v-spacer>

      <!-- Navigation -->
      <v-btn
        icon
        small
        :disabled="!opendedEmailMeta.hasPreviousEmail"
        @click="$emit('change-opened-email', 'previous')"
      >
        <v-icon size="28">
          {{ icons.mdiChevronLeft }}
        </v-icon>
      </v-btn>
      <v-btn
        icon
        small
        :disabled="!opendedEmailMeta.hasNextEmail"
        @click="$emit('change-opened-email', 'next')"
      >
        <v-icon size="28">
          {{ icons.mdiChevronRight }}
        </v-icon>
      </v-btn>
    </div>

    <v-divider></v-divider>

    <!-- 2nd Row -->
    <div class="d-flex align-center px-sm-6 px-4 py-2">
      <v-btn
        v-show="$route.params.folder !== 'trash'"
        icon
        small
        class="me-2"
        @click="$emit('move-email-to-folder', 'trash')"
      >
        <v-icon size="22">
          {{ icons.mdiTrashCanOutline }}
        </v-icon>
      </v-btn>
      <v-btn
        icon
        small
        class="me-2"
        @click="$emit('mark-email-unread')"
      >
        <v-icon size="22">
          {{ icons.mdiEmailOutline }}
        </v-icon>
      </v-btn>
      <v-menu
        offset-y
        min-width="140"
      >
        <template #activator="{ on, attrs }">
          <v-btn
            icon
            small
            class="me-2"
            v-bind="attrs"
            v-on="on"
          >
            <v-icon size="22">
              {{ icons.mdiFolderOutline }}
            </v-icon>
          </v-btn>
        </template>
        <v-list>
          <v-list-item
            v-for="folder in moveToFolderMenuListItems($route)"
            :key="folder.title"
            link
            @click="$emit('move-email-to-folder', folder.value)"
          >
            <v-list-item-icon>
              <v-icon
                size="20"
                class="me-2"
              >
                {{ folder.icon }}
              </v-icon>
            </v-list-item-icon>
            <v-list-item-content>
              <v-list-item-title>{{ folder.title }}</v-list-item-title>
            </v-list-item-content>
          </v-list-item>
        </v-list>
      </v-menu>
      <v-menu
        offset-y
        min-width="140"
      >
        <template #activator="{ on, attrs }">
          <v-btn
            icon
            small
            class="me-2"
            v-bind="attrs"
            v-on="on"
          >
            <v-icon size="22">
              {{ icons.mdiLabelOutline }}
            </v-icon>
          </v-btn>
        </template>
        <v-list>
          <v-list-item
            v-for="label in updateLabelMenuListItems"
            :key="label.title"
            link
            @click="$emit('update-email-label', label.value)"
          >
            <v-list-item-icon class="align-self-center">
              <v-badge
                inline
                :color="label.color"
                dot
                class="mb-1"
              ></v-badge>
            </v-list-item-icon>
            <v-list-item-content>
              <v-list-item-title>{{ label.title }}</v-list-item-title>
            </v-list-item-content>
          </v-list-item>
        </v-list>
      </v-menu>

      <v-spacer></v-spacer>

      <v-btn
        v-if="emailViewData.replies && emailViewData.replies.length"
        icon
        small
        class="me-2"
        @click="showWholeThread = !showWholeThread"
      >
        <v-icon size="22">
          {{ showWholeThread ? icons.mdiArrowCollapseVertical : icons.mdiArrowExpandVertical }}
        </v-icon>
      </v-btn>

      <v-menu
        offset-y
        min-width="140"
      >
        <template #activator="{ on, attrs }">
          <v-btn
            icon
            small
            v-bind="attrs"
            v-on="on"
          >
            <v-icon size="22">
              {{ icons.mdiDotsVertical }}
            </v-icon>
          </v-btn>
        </template>
        <v-list>
          <v-list-item link>
            <v-list-item-content>
              <v-list-item-title>Create Event</v-list-item-title>
            </v-list-item-content>
          </v-list-item>
          <v-list-item link>
            <v-list-item-content>
              <v-list-item-title>Mute</v-list-item-title>
            </v-list-item-content>
          </v-list-item>
          <v-list-item link>
            <v-list-item-content>
              <v-list-item-title>Forward All</v-list-item-title>
            </v-list-item-content>
          </v-list-item>
        </v-list>
      </v-menu>
    </div>

    <v-divider></v-divider>

    <perfect-scrollbar
      ref="psEmailViewContent"
      :settings="perfectScrollbarSettings"
      class="ps-email-view-content"
    >
      <!-- View earlier messages -->
      <p
        v-if="!showWholeThread && emailViewData.replies && emailViewData.replies.length"
        class="text-base text--primary my-4 cursor-pointer text-center"
      >
        <span
          class="view-all-toggler "
          @click="showWholeThread = true"
        >View {{ emailViewData.replies.length }} Earlier Message{{ emailViewData.replies.length > 1 ? 's' : '' }}</span>
      </p>

      <!-- Replies -->
      <!-- Earlier Email Messages -->
      <div
        class="position-relative ma-sm-7 ma-4"
        :style="messagesWrapperStyles"
      >
        <div
          v-if="!showWholeThread && emailViewData.replies && emailViewData.replies.length"
          class="email-replies-preview-container"
        >
          <div
            v-for="(replyPreview, replyPreviewIndex) in emailViewData.replies.length"
            :key="replyPreview"
            :style="threadStyles(replyPreviewIndex, emailViewData.replies.length)"
            class="email-reply-preview bg-card"
          ></div>
        </div>
        <template v-if="showWholeThread">
          <email-message-card
            v-for="threadMail in emailViewData.replies.slice().reverse()"
            :key="threadMail.id"
            :message="threadMail"
            class="mb-4"
          />
        </template>
        <email-message-card
          :message="emailViewData"
          style="top:0;z-index:10"
        />
        <v-card
          outlined
          class="mt-4"
        >
          <v-card-text class="pa-4">
            <p class="mb-0 text--primary font-weight-medium text-base">
              <span>Click here </span>
              <span class="primary--text cursor-pointer">to Reply</span>
              <span> or </span>
              <span class="primary--text cursor-pointer">Foward</span>
            </p>
          </v-card-text>
        </v-card>
      </div>
    </perfect-scrollbar>
  </div>
</template>

<script>
import {
  mdiChevronLeft,
  mdiChevronRight,
  mdiTrashCanOutline,
  mdiEmailOutline,
  mdiFolderOutline,
  mdiLabelOutline,
  mdiArrowExpandVertical,
  mdiArrowCollapseVertical,
  mdiDotsVertical,
} from '@mdi/js'
// eslint-disable-next-line object-curly-newline
import { computed, nextTick, ref, watch } from '@vue/composition-api'
import { PerfectScrollbar } from 'vue2-perfect-scrollbar'
import { psScrollToBottom, psScrollToTop } from '@core/utils'
import useEmail from './useEmail'

// Local Components
import EmailMessageCard from './EmailMessageCard.vue'

export default {
  components: {
    PerfectScrollbar,

    // Local Components
    EmailMessageCard,
  },
  props: {
    emailViewData: {
      type: Object,
      required: true,
    },
    opendedEmailMeta: {
      type: Object,
      required: true,
    },
  },
  setup(props) {
    // ————————————————————————————————————
    //* ——— useEmail
    // ————————————————————————————————————

    const { resolveLabelColor, moveToFolderMenuListItems, updateLabelMenuListItems } = useEmail()

    // ————————————————————————————————————
    //* ——— Threading
    // ————————————————————————————————————

    const psEmailViewContent = ref(null)
    const psEmailViewContentScrollToBottom = psScrollToBottom(psEmailViewContent)
    const psEmailViewContentScrollToTop = psScrollToTop(psEmailViewContent)
    const showWholeThread = ref(false)

    // Reset expanded thread when email change
    watch(
      () => props.emailViewData.id,
      () => {
        showWholeThread.value = false
      },
    )

    // Scroll to bottom when all emails are expanded
    watch(showWholeThread, val => {
      nextTick(() => {
        if (val) psEmailViewContentScrollToBottom()
        else psEmailViewContentScrollToTop()
      })
    })

    // Messages wrapper styles
    const messagesWrapperStyles = computed(() => {
      if (!showWholeThread.value && props.emailViewData.replies && props.emailViewData.replies.length) {
        return {
          marginTop: `${(props.emailViewData.replies.length + 2) * 4}px !important`,
        }
      }

      return null
    })

    // Style for replies preview (UI)
    const threadStyles = computed(() => (replyIndex, total) => {
      const styles = {}
      const reverseIndex = total - replyIndex - 1

      styles.opacity = 1 - (reverseIndex + 1) * 0.25
      styles.width = `${100 - (reverseIndex + 1) * 5}%`

      return styles
    })

    // ————————————————————————————————————
    //* ——— Perfect Scrollbar
    // ————————————————————————————————————

    const perfectScrollbarSettings = {
      maxScrollbarLength: 60,
      wheelPropagation: false,
    }

    return {
      // useEmail
      resolveLabelColor,
      moveToFolderMenuListItems,
      updateLabelMenuListItems,

      // Threading
      psEmailViewContent,
      showWholeThread,
      threadStyles,
      messagesWrapperStyles,

      // Perfect Scrollbar
      perfectScrollbarSettings,

      // Icons
      icons: {
        mdiChevronLeft,
        mdiChevronRight,
        mdiTrashCanOutline,
        mdiEmailOutline,
        mdiFolderOutline,
        mdiLabelOutline,
        mdiArrowExpandVertical,
        mdiArrowCollapseVertical,
        mdiDotsVertical,
      },
    }
  },
}
</script>

<style lang="scss">
@import '~@core/preset/preset/mixins.scss';
@import '~vuetify/src/components/VCard/_variables.scss';

@include theme--child(ps-email-view-content) using ($material) {
  background-color: rgba(map-deep-get($material, 'primary-shade'), 0.04);
}
.ps-email-view-content {
  height: calc(100% - 46px - 46px - 2px);
  .view-all-toggler {
    &:hover {
      color: var(--v-primary-base) !important;
    }
  }

  .email-replies-preview-container {
    .email-reply-preview {
      height: 15px;
      transform-origin: top;
      margin-left: auto;
      margin-right: auto;

      border-top-left-radius: $card-border-radius;
      border-top-right-radius: $card-border-radius;
      @at-root {
        @include theme--child(ps-email-view-content) using ($material) {
          .email-replies-preview-container {
            .email-reply-preview {
              border-left: thin solid rgba(map-deep-get($material, 'primary-shade'), 0.14);
              border-right: thin solid rgba(map-deep-get($material, 'primary-shade'), 0.14);
              border-top: thin solid rgba(map-deep-get($material, 'primary-shade'), 0.14);
            }
          }
          background-color: rgba(map-deep-get($material, 'primary-shade'), 0.04);
        }
      }
    }
  }
}
</style>
