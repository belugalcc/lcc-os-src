<script lang="ts">
	import { onMount } from 'svelte';
	import type { DockApp } from 'src/types';
	import DockItem from './DockItem.svelte';

	let apps: DockApp[] = [];
	let mouseX: number | null = null;

	const fallbackApps: DockApp[] = [
		{ id: 'finder', name: 'Finder', url: 'https://www.apple.com/macos', imageUrl: '/app-icons/finder/256.png' },
		{ id: 'calendar', name: 'Calendar', url: 'https://calendar.google.com', imageUrl: '/app-icons/calendar/256.png' },
		{ id: 'facetime', name: 'FaceTime', url: 'https://apps.apple.com/us/app/facetime/id1110145091', imageUrl: '/app-icons/facetime/256.png' },
		{ id: 'launchpad', name: 'Launchpad', url: 'https://www.apple.com/macos', imageUrl: '/app-icons/launchpad/256.png' },
		{ id: 'mail', name: 'Mail', url: 'https://mail.google.com', imageUrl: '/app-icons/mail/256.png' },
		{ id: 'maps', name: 'Maps', url: 'https://maps.google.com', imageUrl: '/app-icons/maps/256.png' },
		{ id: 'messages', name: 'Messages', url: 'https://messages.google.com/web', imageUrl: '/app-icons/messages/256.png' },
		{ id: 'photos', name: 'Photos', url: 'https://photos.google.com', imageUrl: '/app-icons/photos/256.png' },
		{ id: 'purus-twitter', name: 'Purus Twitter', url: 'https://x.com', imageUrl: '/app-icons/purus-twitter/256.png' },
		{ id: 'safari', name: 'Safari', url: 'https://www.apple.com/safari', imageUrl: '/app-icons/safari/256.png' },
		{ id: 'system-preferences', name: 'System Preferences', url: 'https://support.apple.com/guide/mac-help/welcome/mac', imageUrl: '/app-icons/system-preferences/256.png' },
		{ id: 'view-source', name: 'View Source', url: 'view-source:https://example.com', imageUrl: '/app-icons/view-source/256.png' },
	];

	function parseAppList(text: string): DockApp[] {
		const parsed = text
			.split('\n')
			.map((line) => line.trim())
			.filter((line) => line.length > 0)
			.map((line) => {
				const idMatch = line.match(/^([\w-]+):/);
				const nameMatch = line.match(/name\s*=\s*"([^"]+)"/i);
				const urlMatch = line.match(/URL\s*=\s*"([^"]+)"/i);
				const imageMatch = line.match(/imgeurl\s*=\s*"([^"]+)"/i);

				if (!idMatch || !nameMatch || !urlMatch || !imageMatch) {
					return null;
				}

				return {
					id: idMatch[1],
					name: nameMatch[1],
					url: urlMatch[1],
					imageUrl: imageMatch[1],
				} satisfies DockApp;
			})
			.filter((app): app is DockApp => app !== null);

		return parsed.length > 0 ? parsed : fallbackApps;
	}

	onMount(async () => {
		try {
			const response = await fetch('/app-list.txt');
			if (!response.ok) {
				apps = fallbackApps;
				return;
			}
			const appListText = await response.text();
			apps = parseAppList(appListText);
		} catch {
			apps = fallbackApps;
		}
	});
</script>

<section class="dock-container">
	<div
		class="dock-el"
		on:mousemove={(event) => (mouseX = event.x)}
		on:mouseleave={() => (mouseX = null)}
	>
		{#each apps as app, index}
			<DockItem {mouseX} {app} {index} />
		{/each}
	</div>
</section>

<style>
	.dock-container {
		margin-bottom: 0.3rem;
		left: 0;
		bottom: 0;
		z-index: 9900;
		position: fixed;
		width: 100%;
		height: 5rem;
		padding: 0.4rem;
		display: flex;
		justify-content: center;
	}

	.dock-el {
		backdrop-filter: blur(5px);
		background-color: hsla(240, 3%, 11%, 0.4);
		box-shadow:
			inset 0 0 0 0.2px rgb(245 245 245 / 70%),
			hsla(0, 0%, 0%, 0.3) 2px 5px 19px 7px;
		padding: 0.3rem;
		border-radius: 1.2rem;
		height: 100%;
		display: flex;
		align-items: flex-end;
		animation: dock-bounce-in 900ms cubic-bezier(0.15, 1.25, 0.2, 1) 900ms both;
	}

	@keyframes dock-bounce-in {
		0% {
			transform: translateY(120px) scaleX(0.94);
		}
		60% {
			transform: translateY(-16px) scaleX(1.03);
		}
		80% {
			transform: translateY(6px) scaleX(0.99);
		}
		100% {
			transform: translateY(0) scaleX(1);
		}
	}
</style>
