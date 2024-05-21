<template>
	<div class="contents">
		<div class="form-wrapper form-wrapper-sm">
			<PageHeader>Table</PageHeader>
			<form @submit.prevent="registerUser" class="form">
				<div>
					<div class="button-container">
						<button @click="addRow" type="button">+</button>
						<button @click="removeRow" type="button">-</button>
						<input
							type="file"
							@change="uploadExcel"
							accept=".xlsx, .xls"
							class="upload-btn"
						/>
					</div>
					<table class="styled-table">
						<thead>
							<tr>
								<th>
									<input
										type="checkbox"
										@change="toggleAll"
										:checked="allChecked"
									/>
								</th>
								<th>Etc1</th>
								<th>Etc2</th>
								<th>Amt</th>
							</tr>
						</thead>
						<tbody>
							<tr v-for="(item, index) in items" :key="index">
								<td><input type="checkbox" v-model="item.checked" /></td>
								<td>
									<input
										type="text"
										v-model="item.etc1"
										@blur="$v.items.$each[index].etc1.$touch()"
										:class="{
											'input-error': $v.items.$each[index].etc1.$error,
										}"
									/>
									<span
										v-if="
											$v.items.$each[index].etc1.$error &&
												!$v.items.$each[index].etc1.required
										"
										class="error-message"
									>
										필수 입력 항목입니다.
									</span>
									<span
										v-if="
											$v.items.$each[index].etc1.$error &&
												!$v.items.$each[index].etc1.maxLength
										"
										class="error-message"
									>
										길이가 10자리를 넘을 수 없습니다.
									</span>
									<span
										v-if="
											$v.items.$each[index].etc1.$error &&
												!$v.items.$each[index].etc1.notDuplicate
										"
										class="error-message"
									>
										중복된 값이 있습니다.
									</span>
								</td>
								<td>
									<input
										type="text"
										v-model="item.etc2"
										@blur="$v.items.$each[index].etc2.$touch()"
										:class="{
											'input-error': $v.items.$each[index].etc2.$error,
										}"
									/>
									<span
										v-if="
											$v.items.$each[index].etc2.$error &&
												!$v.items.$each[index].etc2.required
										"
										class="error-message"
									>
										필수 입력 항목입니다.
									</span>
								</td>
								<td>
									<input
										type="number"
										v-model="item.amt"
										@blur="$v.items.$each[index].amt.$touch()"
										:class="{ 'input-error': $v.items.$each[index].amt.$error }"
									/>
									<span
										v-if="
											$v.items.$each[index].amt.$error &&
												!$v.items.$each[index].amt.required
										"
										class="error-message"
									>
										필수 입력 항목입니다.
									</span>
									<span
										v-if="
											$v.items.$each[index].amt.$error &&
												!$v.items.$each[index].amt.numeric
										"
										class="error-message"
									>
										숫자여야 합니다.
									</span>
								</td>
							</tr>
						</tbody>
						<tfoot>
							<tr>
								<td colspan="3">Total</td>
								<td>{{ totalAmt }}</td>
							</tr>
						</tfoot>
					</table>
					<div class="button-container">
						<button type="submit" class="submit-button">등록</button>
						<a href="#" @click.prevent="downloadExcel" class="submit-button">
							엑셀 템플릿 다운로드
						</a>
					</div>
				</div>
			</form>
		</div>
	</div>
</template>

<script>
import PageHeader from './common/PageHeader.vue';
import { required, maxLength, numeric } from 'vuelidate/lib/validators';
import * as XLSX from 'xlsx';

export default {
	components: {
		PageHeader,
	},
	data() {
		return {
			items: [
				{ checked: false, etc1: '', etc2: '', amt: 0 },
				// 추가 행을 원하면 여기에 객체를 추가하세요
			],
		};
	},
	validations() {
		const notDuplicate = value => {
			if (!value) return true; // 값이 비어 있으면 중복 검사를 건너뜁니다.
			const etc1Values = this.items.map(item => item.etc1);
			const occurrences = etc1Values.filter(etc1 => etc1 === value).length;
			return occurrences <= 1;
		};

		return {
			items: {
				$each: {
					etc1: {
						required,
						maxLength: maxLength(10),
						notDuplicate,
					},
					etc2: { required },
					amt: { required, numeric },
				},
			},
		};
	},
	computed: {
		allChecked() {
			return this.items.every(item => item.checked);
		},
		totalAmt() {
			return this.items.reduce((sum, item) => sum + Number(item.amt), 0);
		},
	},
	methods: {
		notDuplicate(value) {
			if (!value) return true; // 값이 비어 있으면 중복 검사를 건너뜁니다.
			const etc1Values = this.items.map(item => item.etc1);
			const occurrences = etc1Values.filter(etc1 => etc1 === value).length;
			return occurrences <= 1;
		},
		registerUser() {
			this.$v.$touch();
			if (this.$v.$invalid) {
				alert('입력값을 확인해주세요.');
				return;
			}
			// 유효성 검사를 통과한 경우 폼 데이터를 처리합니다.
		},
		toggleAll(event) {
			const isChecked = event.target.checked;
			this.items.forEach(item => {
				item.checked = isChecked;
			});
		},
		addRow() {
			this.items.push({ checked: false, etc1: '', etc2: '', amt: 0 });
		},
		removeRow() {
			if (this.items.length > 1) {
				this.items.pop();
			}
		},
		downloadExcel() {
			// 서버로 GET 요청을 보내고 응답을 처리하여 파일을 다운로드
			console.log(`${process.env.VUE_APP_API_URL}download/excel`);
			fetch(`${process.env.VUE_APP_API_URL}download/excel`, {
				method: 'GET',
				headers: {
					'Content-Type': 'application/octet-stream',
				},
			})
				.then(response => response.blob())
				.then(blob => {
					const url = window.URL.createObjectURL(new Blob([blob]));
					const link = document.createElement('a');
					link.href = url;
					link.setAttribute('download', 'invoceTemplate.xlsx');
					document.body.appendChild(link);
					link.click();
					link.parentNode.removeChild(link);
				})
				.catch(error => console.error('Error downloading the file:', error));
		},
		uploadExcel(event) {
			this.items = [];
			const file = event.target.files[0];
			const reader = new FileReader();
			reader.onload = e => {
				const data = new Uint8Array(e.target.result);
				const workbook = XLSX.read(data, { type: 'array' });
				const firstSheetName = workbook.SheetNames[0];
				const worksheet = workbook.Sheets[firstSheetName];
				const jsonData = XLSX.utils.sheet_to_json(worksheet, { header: 1 });

				// 엑셀 데이터의 각 행을 items 배열에 추가
				this.items = jsonData.slice(1).map(row => ({
					checked: !!row[0],
					etc1: row[1] || '',
					etc2: row[2] || '',
					amt: row[3] || 0,
				}));
			};
			reader.readAsArrayBuffer(file);
		},
	},
};
</script>

<style scoped>
.button-container {
	margin: 20px 0;
	display: flex;
	justify-content: center;
}

.button-container button,
.upload-btn {
	padding: 10px 20px;
	margin-right: 5px;
	font-size: 16px;
	cursor: pointer;
	border: none;
	border-radius: 4px;
	transition: background-color 0.3s ease;
}

.add-btn {
	background-color: #28a745;
	color: #ffffff;
}

.add-btn:hover {
	background-color: #218838;
}

.remove-btn {
	background-color: #dc3545;
	color: #ffffff;
}

.remove-btn:hover {
	background-color: #c82333;
}

.upload-btn {
	background-color: #007bff;
	color: #ffffff;
}

.upload-btn:hover {
	background-color: #0056b3;
}

.styled-table {
	width: 100%;
	border-collapse: collapse;
	margin: 25px 0;
	font-size: 18px;
	text-align: left;
}

.styled-table thead tr {
	background-color: #009879;
	color: #ffffff;
	text-align: left;
	font-weight: bold;
}

.styled-table th,
.styled-table td {
	padding: 12px 15px;
	border: 1px solid #dddddd;
}

.styled-table tbody tr {
	border-bottom: 1px solid #dddddd;
}

.styled-table tbody tr:nth-of-type(even) {
	background-color: #f3f3f3;
}

.styled-table tbody tr:last-of-type {
	border-bottom: 2px solid #009879;
}

.styled-table tfoot tr {
	background-color: #f1f1f1;
	font-weight: bold;
}

.styled-table tfoot td {
	padding: 12px 15px;
}

.input-error {
	border: 2px solid red;
}

.error-message {
	color: red;
	font-size: 14px;
	display: block;
	margin-top: 5px;
}

.submit-button {
	padding: 10px 20px;
	font-size: 16px;
	cursor: pointer;
	border: none;
	border-radius: 4px;
	background-color: #007bff;
	color: #ffffff;
	transition: background-color 0.3s ease;
}

.submit-button:hover {
	background-color: #0056b3;
}
</style>
