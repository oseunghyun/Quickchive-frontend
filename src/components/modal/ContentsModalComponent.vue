<!-- 콘텐츠 추가 모달 -->
<template>
  <div class="modal">
    <div class="overlay"></div>
    <div class="contents-modal-card">
      <div class="modal-card__header">
        <h1>콘텐츠 추가</h1>
        <button
          type="button"
          class="btn--transparent btn__close"
          @click="$emit('close-modal')"
        >
          <img :src="closeBtn" />
        </button>
      </div>
      <div class="modal-card__wrapper">
        <div class="register-form__wrapper">
          <label class="register-form__label">링크<em>*</em></label>
          <div class="flex-container modal-form__wrapper">
            <input
              v-model="link"
              placeholder="URL 입력"
              oninput="this.value = this.value.replace(/ +/g, ' ')"
            />
            <button
              @click="addFavorites()"
              class="btn--transparent btn__favorites"
            >
              <img v-show="!favorite" :src="star_border" />
              <img v-show="favorite" :src="star" />
            </button>
          </div>
        </div>
        <div class="flex-container-col">
          <button
            v-show="!isDetailSettingActive"
            class="btn__deleteuser"
            @click="setDetail"
          >
            세부 설정
          </button>
        </div>
        <!-- 세부 설정 선택 시 나오는 인풋 -->
        <div v-show="isDetailSettingActive">
          <div class="register-form__wrapper">
            <label class="register-form__label">이름</label>
            <input v-model="title" placeholder="30자 이하 권장" />
          </div>
          <div class="flex-container modal-form__wrapper">
            <div class="register-form__wrapper">
              <label class="register-form__label">카테고리</label>
              <select v-model="categoryName" class="contents-modal__select">
                <option value="" selected>카테고리 선택</option>
                <option v-for="(category, index) in myCategories" :key="index">
                  {{ category.name }}
                </option>
              </select>
            </div>
            <div class="register-form__wrapper">
              <label class="register-form__label">읽을 기한</label>
              <div class="flex-container modal-form__wrapper">
                <input v-model="deadline" type="date" />
                <span class="contents-modal__date-description"
                  >D-DAY에 알림을 보내드립니다</span
                >
              </div>
            </div>
          </div>
          <div class="register-form__wrapper">
            <label class="register-form__label">메모</label>
            <input v-model="comment" placeholder="500자 이하" maxlength="500" />
          </div>
        </div>
      </div>

      <!-- 콜렉션 안내 문구 및 버튼 -->
      <div class="modal-card__collection__wrapper">
        <span class="modal-card__text"
          ><img :src="alert_circle" />여러 링크들을 묶어서 저장하고
          싶다면?</span
        >
        <button
          @click="openCollectionModal()"
          class="btn--transparent btn__collection"
        >
          콜렉션으로 저장하기 >>
        </button>
      </div>
      <div class="flex-container-col">
        <button
          :disabled="!link"
          @click="createContent()"
          class="btn--sm btnPrimary"
        >
          저장
        </button>
      </div>
    </div>
    <AlertModalComponent
      v-if="isAlertModalActive == true"
      :alertModalContent="alertModalContent"
      :btnMessage="btnMessage"
      @confirmBtn="isAlertModalActive = false"
    ></AlertModalComponent>
  </div>
</template>

<script>
import closeBtn from "@/assets/icon/closeBtn.svg";
import star_border from "@/assets/icon/star_border.svg";
import star from "@/assets/icon/star.svg";
import alert_circle from "@/assets/icon/alert-circle.svg";
import { addContents } from "@/api/contents";
import { validateLink, linkCounter, filterLink } from "@/utils/validation";
import AlertModalComponent from "@/components/modal/AlertModalComponent.vue";

export default {
  components: { AlertModalComponent },
  data() {
    return {
      closeBtn,
      star_border,
      star,
      alert_circle,
      isDetailSettingActive: false,
      // 내 카테고리 목록
      myCategories: {},
      // 폼 항목
      link: "",
      title: "",
      deadline: "",
      comment: "",
      categoryName: "",
      favorite: false,
      data: {},
      linkList: [],
      // 경고 모달
      isAlertModalActive: false,
      AlertModalContent: "",
      btnMessage: "네",
    };
  },
  async created() {
    await this.$store.dispatch("GET_CATEGORIES");
    this.myCategories = this.$store.getters.getCategories;
  },
  props: {
    contentsData: Object,
  },
  watch: {
    link: function() {
      this.linkList = filterLink(this.link);
    },
  },
  computed: {
    // 링크 여부 확인
    isTextLink() {
      if (this.link != "") {
        return validateLink(this.link);
      } else {
        return null;
      }
    },
    // 링크 개수 확인
    countLink() {
      if (this.link != "") {
        return linkCounter(this.link);
      } else {
        return null;
      }
    },
  },
  methods: {
    setDetail() {
      this.isDetailSettingActive = true;
    },
    addFavorites() {
      this.favorite = !this.favorite;
    },

    // 콘텐츠 추가 요청
    async createContent() {
      // 링크가 1개인 경우
      if (this.countLink == 1) {
        const contentsData = {
          link: this.link,
          favorite: this.favorite,
          comment: this.comment,
          deadline: this.deadline,
          categoryName: this.categoryName,
          title: this.title,
        };
        Object.keys(contentsData).forEach(
          (key) =>
            (contentsData[key] == "" || contentsData[key] == undefined) &&
            delete contentsData[key]
        );
        {
          try {
            await addContents(contentsData);
            this.$emit("close-modal");
          } catch (error) {
            this.alertModalContent = error.response.data.message;
            this.isAlertModalActive = true;
          }
        }
      }
      // 링크가 2개 이상인 경우
      else if (this.countLink >= 2) {
        const linkList = this.linkList;
        this.$emit("isLinkNotSingle", linkList);
      }
    },
    openCollectionModal() {
      const linkList = this.linkList;
      this.$emit("openCollectionModal", linkList);
    },
  },
};
</script>

<style></style>
