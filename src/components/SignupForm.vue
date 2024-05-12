<template>
	<div class="contents">
		<div class="form-wrapper form-wrapper-sm">
			<PageHeader>Sign Up</PageHeader>
			<form @submit.prevent="registerUser" class="form">
				<!-- ID -->
				<div>
					<label for="username">ID</label>
					<input
						type="text"
						id="username"
						v-model="username"
						:class="usernameValidClass"
					/>
					<p class="validation-text">
						<span class="warning" v-if="!isUsernameValid">
							Please enter an email address
						</span>
					</p>
				</div>
				<!-- PW -->
				<div>
					<label for="password">PW</label>
					<input
						type="password"
						id="password"
						v-model="password"
						:class="passwordValidClass"
					/>
					<p class="validation-text">
						<span class="warning" v-if="!isPasswordValid">
							Password must be over 8 letters
						</span>
					</p>
				</div>
				<!-- Nickname -->
				<div>
					<label for="nickname">Nickname</label>
					<input
						type="text"
						id="nickname"
						v-model="nickname"
						:class="nicknameValidClass"
					/>
				</div>
				<!-- Attatch -->
				<div>
					<label for="username">Attatch</label>
					<div
						v-for="(fileInput, index) in fileInputs"
						:key="index"
						class="file-input-container"
					>
						<input
							type="file"
							@change="validateFile(index, $event)"
							class="file-input"
							ref="files"
						/>
						<!-- 첫 번째 입력 필드에만 버튼 표시 -->
						<div v-if="index === 0" class="button-container">
							<button v-if="fileInputs.length < 5" @click="addInput">+</button>
							<button
								v-if="fileInputs.length > 1"
								@click="removeInput"
								class="remove-button"
							>
								-
							</button>
						</div>
					</div>
				</div>
				<button
					type="submit"
					class="btn"
					:class="isButtonDisabled"
					:disabled="isButtonDisabled"
				>
					Create
				</button>
			</form>
			<p class="log">
				{{ logMessage }}
			</p>
		</div>
	</div>
</template>

<script>
import { signupUser } from '@/api/auth';
import { validateEmail, validatePassword } from '@/utils/validation';
import PageHeader from './common/PageHeader.vue';

export default {
	components: {
		PageHeader,
	},
	data() {
		return {
			username: '',
			password: '',
			nickname: '',
			logMessage: '',
			fileInputs: [{ error: '' }], // 초기 값으로 오류 메시지 필드를 포함
		};
	},
	computed: {
		usernameValidClass() {
			if (!this.username) {
				return;
			}
			return validateEmail(this.username) ? 'valid' : 'invalid';
		},
		isUsernameValid() {
			if (!this.username) {
				return true;
			}
			return validateEmail(this.username);
		},
		passwordValidClass() {
			if (!this.password) {
				return;
			}
			return validatePassword(this.password) ? 'valid' : 'invalid';
		},
		isPasswordValid() {
			if (!this.password) {
				return true;
			}
			return validatePassword(this.password);
		},
		nicknameValidClass() {
			return this.nickname ? 'valid' : null;
		},
		isButtonDisabled() {
			return !this.username ||
				!this.password ||
				!this.nickname ||
				!validateEmail(this.username) ||
				!validatePassword(this.password)
				? 'disabled'
				: null;
		},
	},
	methods: {
		async registerUser() {
			try {
				const formData = new FormData();
				this.fileInputs.forEach(fileInput => {
					if (fileInput) {
						formData.append('files', fileInput); // 모든 파일을 'files'로 추가
					}
				});
				console.log('formData :::', formData);

				await signupUser({
					username: this.username,
					password: this.password,
					nickname: this.nickname,
				});
				this.logMessage = 'User is created';
				this.initForm();
			} catch (error) {
				console.log(error.response);
				if (error.response.status === 409) {
					this.logMessage = `${this.username} already exists`;
				}
			}
		},
		initForm() {
			this.username = '';
			this.password = '';
			this.nickname = '';
		},
		addInput() {
			if (this.fileInputs.length < 5) {
				this.fileInputs.push({}); // 새 파일 입력 필드 추가
			}
		},
		removeInput() {
			if (this.fileInputs.length > 1) {
				this.fileInputs.splice(1, 1); // 두 번째 입력 필드 제거
			}
		},
		validateFile(index, event) {
			const file = event.target.files[0];
			const fileInput = this.fileInputs[index];
			if (file) {
				if (!['image/jpeg', 'image/png'].includes(file.type)) {
					fileInput.error = 'JPEG 또는 PNG 파일만 업로드 가능합니다.';
					event.target.value = ''; // 파일 입력 필드 리셋
				} else if (file.size > 2 * 1024 * 1024) {
					// 2MB 제한
					fileInput.error = '파일 크기는 2MB를 초과할 수 없습니다.';
					event.target.value = ''; // 파일 입력 필드 리셋
				} else {
					fileInput.error = ''; // 유효성 검사 통과
					this.$set(this.fileInputs, index, event.target.files[0]); // 선택된 파일 저장
				}
			}
		},
	},
};
</script>

<style scoped>
.form {
	width: 460px;
	height: 100%;
}
.form .validation-text {
	margin-top: -0.5rem;
	margin-bottom: 0.5rem;
	font-size: 1rem;
	display: flex;
	flex-direction: row-reverse;
	justify-content: space-between;
}
.btn {
	color: white;
}
.log {
	width: 460px;
}
.container {
	display: flex;
}

.container {
	display: flex;
	flex-direction: column;
}

.file-input-container {
	display: flex;
	align-items: center;
	margin-bottom: 10px;
}

.file-input {
	flex-grow: 1;
	/* 입력 필드가 가능한 모든 공간을 채움 */
	margin-right: 10px;
	/* 입력 필드와 버튼 사이 간격 */
}

.button-container {
	display: flex;
	align-items: center;
}

button {
	padding: 5px 10px;
	color: white;
	background-color: #4caf50;
	/* 녹색 배경 */
	border: none;
	cursor: pointer;
	margin-left: 5px;
}

.remove-button {
	background-color: #f44336;
	/* 빨간색 배경 */
}
</style>
