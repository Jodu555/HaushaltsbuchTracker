<template>
	<header>
		<nav class="navbar navbar-expand-lg bg-body-tertiary">
			<div class="container-fluid">
				<a class="navbar-brand" href="#">Haushaltsbuch</a>
				<button
					class="navbar-toggler"
					type="button"
					data-bs-toggle="collapse"
					data-bs-target="#navbarNav"
					aria-controls="navbarNav"
					aria-expanded="false"
					aria-label="Toggle navigation">
					<span class="navbar-toggler-icon"></span>
				</button>
				<div class="collapse navbar-collapse" id="navbarNav">
					<ul class="navbar-nav">
						<li class="nav-item">
							<NuxtLink to="/" class="nav-link" active-class="active">Home</NuxtLink>
						</li>
						<li class="nav-item">
							<NuxtLink to="/view" class="nav-link" active-class="active">View</NuxtLink>
						</li>
					</ul>
				</div>
			</div>
		</nav>
	</header>
	<main class="container">
		<slot />
	</main>
	<footer>
		<!-- place footer here -->
	</footer>
</template>

<script lang="ts" setup></script>

<style scoped></style>
