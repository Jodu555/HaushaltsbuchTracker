<template>
	<div>
		<h1 class="text-center mt-2 mb-4">Previous List</h1>
		<div class="d-flex justify-content-center">
			<div class="mb-3 col-5">
				<label for="sliceAmount" class="form-label">Display Amount</label>
				<input type="number" class="form-control" v-model="sliceAmount" id="sliceAmount" aria-describedby="helpId" placeholder="" />
				<small id="helpId" class="form-text text-muted">The Maximum amount of items to display</small>
			</div>
		</div>

		<div v-for="active in activeData" class="mt-3" :key="active.slug">
			<label class="form-label h2" :for="active.slug">Active {{ active.slug }}</label>
			<div v-if="previous?.[active.slug] == undefined" class="d-flex justify-content-center">
				<div class="spinner-border text-primary" role="status">
					<span class="visually-hidden">Loading...</span>
				</div>
			</div>
			<div v-else class="table-responsive">
				<table class="table">
					<thead>
						<tr>
							<th scope="col">Date</th>
							<th scope="col">Value</th>
						</tr>
					</thead>
					<tbody>
						<tr class="" v-for="item in previous?.[active.slug].slice(0, sliceAmount)" :key="item.x">
							<td scope="row">{{ new Date(item.x).toLocaleString() }}</td>
							<td>{{ item.y }}</td>
						</tr>
					</tbody>
				</table>
			</div>
		</div>
	</div>
</template>

<script lang="ts" setup>
const API_URL = 'http://192.168.178.23:4444/api';
// const API_URL = 'http://localhost:3600/api';

const sliceAmount = ref(10);

const {
	data: activeData,
	status: activeStatus,
	error: activeError,
	refresh: activeRefresh,
} = await useFetch(`${API_URL}/active`, {
	transform: (data: string[]) =>
		data.map((slug) => {
			return { slug, input: '' };
		}),
});

const previous = computedAsync(async () => {
	const output: Record<string, { x: number; y: number }[]> = {};
	for (const active of activeData.value!) {
		const lastF = (await fetch(`${API_URL}/data/${active.slug}/true`).then((res) => res.json())) as { x: number; y: number }[];
		lastF.sort((a, b) => b.x - a.x);
		output[active.slug] = lastF;
	}
	return output;
});
</script>

<style scoped></style>
