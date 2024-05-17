<template>
	<div class="contents">
		<div class="form-wrapper form-wrapper-sm">
			<PageHeader>Table</PageHeader>
			<form @submit.prevent="registerUser" class="form">
				<div>
					<div class="button-container">
						<button @click="addRow">+</button>
						<button @click="removeRow">-</button>
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
										:class="{ 'input-error': item.etc1.length > 10 }"
									/>
									<span v-if="item.etc1.length > 10" class="error-message"
										>길이가 10자리가 넘어갑니다</span
									>
								</td>
								<td><input type="text" v-model="item.etc2" /></td>
								<td><input type="number" v-model="item.amt" /></td>
							</tr>
						</tbody>
						<tfoot>
							<tr>
								<td colspan="3">Total</td>
								<td>{{ totalAmt }}</td>
							</tr>
						</tfoot>
					</table>
				</div>
			</form>
			<p class="log">
				{{ logMessage }}
			</p>
		</div>
	</div>
</template>

<script>
import PageHeader from './common/PageHeader.vue';
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
	computed: {
		allChecked() {
			return this.items.every(item => item.checked);
		},
		totalAmt() {
			return this.items.reduce((sum, item) => sum + Number(item.amt), 0);
		},
	},
	methods: {
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
		uploadExcel(event) {
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
	margin-bottom: 10px;
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
	/* 녹색 */
	color: #ffffff;
}

.add-btn:hover {
	background-color: #218838;
}

.remove-btn {
	background-color: #dc3545;
	/* 빨간색 */
	color: #ffffff;
}

.remove-btn:hover {
	background-color: #c82333;
}

.upload-btn {
	background-color: #007bff;
	/* 파란색 */
	color: #ffffff;
}

.upload-btn:hover {
	background-color: #0056b3;
}

.download-btn {
	background-color: #ffc107;
	/* 노란색 */
	color: #ffffff;
}

.download-btn:hover {
	background-color: #e0a800;
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
</style>
