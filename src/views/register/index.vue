<template>
  <div class="common-layout">
    <el-container class="app_wrapper">
      <el-main class="register_wrapper">
        <el-form
          class="register_form"
          :model="registerForm"
          :rules="currentRules"
          ref="registerForms"
        >
          <h1>注册</h1>
          <h2>欢迎加入药品甄选</h2>

          <!-- 注册类型选择 -->
          <el-radio-group v-model="registerType" class="register-type">
            <el-radio-button label="普通用户">普通用户</el-radio-button>
            <el-radio-button label="医师">医师</el-radio-button>
          </el-radio-group>

          <el-form-item prop="userName">
            <el-input
              :prefix-icon="User"
              v-model.trim="registerForm.userName"
              placeholder="用户名"
            ></el-input>
          </el-form-item>

          <el-form-item prop="userAccount">
            <el-input
              :prefix-icon="User"
              v-model.trim="registerForm.userAccount"
              placeholder="账号"
            ></el-input>
          </el-form-item>

          <el-form-item prop="userPwd">
            <el-input
              type="password"
              :prefix-icon="Lock"
              v-model.trim="registerForm.userPwd"
              show-password
              placeholder="密码"
            ></el-input>
          </el-form-item>

          <el-form-item prop="confirmPassword">
            <el-input
              type="password"
              :prefix-icon="Lock"
              v-model.trim="registerForm.confirmPassword"
              show-password
              placeholder="确认密码"
            ></el-input>
          </el-form-item>

          <el-form-item prop="userEmail">
            <div class="input-group">
              <div>
                <el-input
                  v-model.trim="registerForm.userEmail"
                  placeholder="邮箱"
                  @blur="validateEmail"
                  class="fixed-width"
                ></el-input>
              </div>
              <div>
                <el-button :disabled="isGettingCode" @click="getEmailCode">
                  {{ isGettingCode ? `重新获取验证码 (${countdown})` : "获取邮箱验证码" }}
                </el-button>
              </div>
            </div>
          </el-form-item>

          <el-form-item prop="userSex">
            <el-select
              v-model="registerForm.userSex"
              placeholder="请选择性别"
              class="fixed-width"
            >
              <el-option label="男" value="male"></el-option>
              <el-option label="女" value="female"></el-option>
            </el-select>
          </el-form-item>

          <el-form-item prop="userTel">
            <el-input
              v-model.trim="registerForm.userTel"
              placeholder="电话"
              @blur="validateTel"
              class="fixed-width"
            ></el-input>
          </el-form-item>

          <el-form-item prop="code">
            <el-input v-model.trim="registerForm.code" placeholder="验证码"></el-input>
          </el-form-item>

          <el-form-item prop="userAge">
            <el-input
              v-model.number="registerForm.userAge"
              placeholder="真实年龄"
            ></el-input>
          </el-form-item>

          <!-- 医师注册额外字段 -->
          <el-form-item v-if="registerType === '医师'" prop="medicalLicenseNumber">
            <el-input
              v-model.trim="registerForm.medicalLicenseNumber"
              placeholder="医师执业资格证书号"
            ></el-input>
          </el-form-item>

          <el-form-item v-if="registerType === '医师'" prop="idCard">
            <el-input
              v-model.trim="registerForm.idCard"
              placeholder="身份证号码"
            ></el-input>
          </el-form-item>

          <el-form-item>
            <el-button
              :loading="loading"
              type="primary"
              size="default"
              class="register_btn"
              @click="register"
            >
              注册
            </el-button>
          </el-form-item>
        </el-form>
      </el-main>
    </el-container>
  </div>
</template>

<script setup lang="ts">
import { User, Lock } from "@element-plus/icons-vue";
import { reactive, ref, computed } from "vue";
import { ElMessage } from "element-plus";
import { useRouter } from "vue-router";
import useUserStore from "@/stores/modules/user";
import { reqSendEmailCode, reqRegister } from "@/api/user/index";

const loading = ref(false);
const isGettingCode = ref(false);
const countdown = ref(60);
const $router = useRouter();
const useStore = useUserStore();

const registerType = ref("普通用户"); // 注册类型：普通用户或医师

const registerForm = reactive({
  imgPath: "",
  confirmPassword: "",
  userAccount: "",
  userName: "",
  userPwd: "",
  userAge: "",
  code: "",
  userEmail: "",
  userSex: "",
  userTel: "",
  medicalLicenseNumber: null,
  idCard: null,
});

const avatarImages = [
  "/images/userImg/img_0.png",
  "/images/userImg/img_1.png",
  "/images/userImg/img_2.png",
  "/images/userImg/img_3.png",
  "/images/userImg/img_4.png",
  "/images/userImg/img_5.png",
  "/images/userImg/img_6.png",
  "/images/userImg/img_7.png",
  "/images/userImg/img_8.png",
];

const registerForms = ref();

const startCountdown = () => {
  isGettingCode.value = true;
  countdown.value = 60;
  const interval = setInterval(() => {
    countdown.value -= 1;
    if (countdown.value <= 0) {
      clearInterval(interval);
      isGettingCode.value = false;
    }
  }, 1000);
};

const validateEmail = async () => {
  await registerForms.value.validateField("userEmail");
};

const getEmailCode = async () => {
  if (!registerForm.userEmail) {
    ElMessage.error("请输入邮箱后获取验证码");
    return;
  }

  try {
    const res = await reqSendEmailCode(registerForm.userEmail);
    if (res.code === 200) {
      ElMessage.success("验证码已发送至您的邮箱");
      startCountdown();
    } else {
      ElMessage.error("获取验证码失败，请重试");
    }
  } catch (error) {
    ElMessage.error("获取验证码失败，请重试");
  }
};

const validatorUserAccount = (_rule: any, value: any, callback: any) => {
  if (value.length >= 6) {
    callback();
  } else {
    callback(new Error("账号长度至少6位"));
  }
};

const validatorUserName = (_rule: any, value: any, callback: any) => {
  if (value.length >= 2) {
    callback();
  } else {
    callback(new Error("账号长度至少2位"));
  }
};

const validatorEmail = (_rule: any, value: any, callback: any) => {
  const emailPattern = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (emailPattern.test(value)) {
    callback();
  } else {
    callback(new Error("请输入有效的邮箱地址"));
  }
};

const validateTel = (_rule: any, value: any, callback: any) => {
  const phoneRegex = /^\d{11}$/;
  if (phoneRegex.test(value)) {
    callback();
  } else {
    callback(new Error("请输入有效的电话号码"));
  }
};

const validatorMedicalLicenseNumber = (_rule: any, value: any, callback: any) => {
  // 医师执业资格证书号正则表达式
  //   医师执业资格证书号: 102010100000001 （假设市级发证，县代码为00）
  //   身份证号码: 123456789012345678
  const medicalLicensePattern = /^(\d)(\d{2})(\d{2})(00|[\d]{2})(00|[\d]{2})(\d{6})$/;

  if (!medicalLicensePattern.test(value)) {
    return callback(new Error("请输入有效的医生执业资格证书编码"));
  }

  // 进一步验证证书流水码是否在000001-999999之间
  const certificateSerial = value.slice(9);
  const serialNumber = parseInt(certificateSerial, 10);

  if (serialNumber < 1 || serialNumber > 999999) {
    return callback(new Error("证书流水码必须在000001到999999之间"));
  }

  // 检查由省、自治区、直辖市卫生行政部门发证的情况
  const provinceCode = value.slice(3, 5);
  const cityCode = value.slice(5, 7);
  const countyCode = value.slice(7, 9);

  if (provinceCode !== "00" && cityCode === "00" && countyCode === "00") {
    return callback();
  }

  // 检查由市级卫生行政部门发证的情况
  if (cityCode !== "00" && countyCode === "00") {
    return callback();
  }

  return callback(new Error("证书编码不符合发证规则"));
};

const validatorIdCard = (_rule: any, value: any, callback: any) => {
  const idCardPattern = /(^\d{15}$)|(^\d{18}$)|(^\d{17}(\d|X|x)$)/;
  if (idCardPattern.test(value)) {
    callback();
  } else {
    callback(new Error("请输入有效的身份证号码"));
  }
};

const validatorPassword = (_rule: any, value: any, callback: any) => {
  if (value.length >= 6) {
    callback();
  } else {
    callback(new Error("密码长度至少六位"));
  }
};

const validatorConfirmPassword = (_rule: any, value: any, callback: any) => {
  if (value === registerForm.userPwd) {
    callback();
  } else {
    callback(new Error("两次输入的密码不一致"));
  }
};

const selectRandomAvatar = () => {
  const randomAvatar = avatarImages[Math.floor(Math.random() * avatarImages.length)];
  registerForm.imgPath = randomAvatar;
};

const register = async () => {
  try {
    await registerForms.value.validate();
    loading.value = true;

    selectRandomAvatar();

    // 如果是普通用户注册，清空医师注册字段
    if (registerType.value === "普通用户") {
      registerForm.medicalLicenseNumber = null;
      registerForm.idCard = null;
    }

    console.log("注册信息:", registerForm);
    const res = await reqRegister(registerForm);
    if (res.code == 200 && res.data != null) {
      ElMessage.success("注册成功，欢迎加入");
      $router.push({ path: "/user/login" });
    } else {
      ElMessage.error(res.msg);
    }
  } catch (error) {
    ElMessage.error("注册过程中出现错误，请稍后再试");
  } finally {
    loading.value = false;
  }
};

// 动态规则
const rules = computed(() => {
  if (registerType.value === "普通用户") {
    return {
      userAccount: [{ trigger: "blur", validator: validatorUserAccount }],
      userName: [{ trigger: "blur", validator: validatorUserName }],
      userEmail: [{ trigger: "blur", validator: validatorEmail }],
      userPwd: [{ trigger: "blur", validator: validatorPassword }],
      confirmPassword: [{ trigger: "blur", validator: validatorConfirmPassword }],
      code: [{ required: true, message: "验证码不能为空", trigger: "blur" }],
      userAge: [{ required: true, message: "请输入真实年龄", trigger: "blur" }],
      userSex: [{ required: true, message: "请选择性别", trigger: "blur" }],
      userTel: [{ trigger: "blur", validator: validateTel }],
    };
  } else {
    return {
      userAccount: [{ trigger: "blur", validator: validatorUserAccount }],
      userName: [{ trigger: "blur", validator: validatorUserName }],
      userEmail: [{ trigger: "blur", validator: validatorEmail }],
      userPwd: [{ trigger: "blur", validator: validatorPassword }],
      confirmPassword: [{ trigger: "blur", validator: validatorConfirmPassword }],
      code: [{ required: true, message: "验证码不能为空", trigger: "blur" }],
      userAge: [{ required: true, message: "请输入真实年龄", trigger: "blur" }],
      userSex: [{ required: true, message: "请选择性别", trigger: "blur" }],
      userTel: [{ trigger: "blur", validator: validateTel }],
      medicalLicenseNumber: [
        { trigger: "blur", validator: validatorMedicalLicenseNumber },
      ],
      idCard: [{ trigger: "blur", validator: validatorIdCard }],
    };
  }
});

const currentRules = computed(() => rules.value);
</script>

<style lang="scss">
.common-layout {
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
}

.app_wrapper {
  width: 100%;
  height: 110vh;
  display: flex;

  justify-content: center;
  align-items: center;
  background: url("@/assets/images/background.jpg") no-repeat center center fixed;
  background-size: cover;
}
.el-input {
  width: 260px;
}
.register_wrapper {
  width: 100%;
  max-width: 520px;
  padding: 40px;
  background: url("@/assets/images/register_form.png") no-repeat center center;
  background-size: cover;
  margin-left: 300px; //向右移动
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
  border-radius: 10px;
  display: flex;
  flex-direction: column;
  align-items: center;

  h1,
  h2 {
    color: white;
  }

  h1 {
    font-size: 40px;
  }

  h2 {
    font-size: 20px;
    margin: 20px 0px;
  }

  .register_btn {
    width: 260px;
  }
}

.fixed-width {
  width: 260px;
}

.input-group {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 100%;

  > div {
    flex: 1;
    margin-right: 10px; // 可以根据需要调整间距
  }

  > div:last-child {
    margin-right: 0;
  }
}
</style>
