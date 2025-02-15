<script>
	import { createEventDispatcher, onMount } from 'svelte';
	import { t } from 'svelte-i18n';
	import positionToStyle from '$lib/helpers/cssPosition';
	import { getOutfit } from '$lib/helpers/wish/outfit';
	import { getName } from '$lib/helpers/nameText';
	import Icon from '$lib/components/utility/Icon.svelte';

	export let preview = false;
	export let previewlist = [];
	export let list = [];
	let clientHeight;
	let Share;
	let playSfx;

	const sortByType = (a, b) => {
		if (a.type > b.type) return 1;
		if (b.type > a.type) return -1;
		return 0;
	};

	const sortByName = (a, b) => {
		if (a.name > b.name) return 1;
		if (a.name < b.name) return -1;
		return 0;
	};

	let sortedWish = [];
	const getList = (preview, previewlist) => {
		if (preview) {
			sortedWish = previewlist;
			return;
		}

		const item = (star) =>
			list
				.filter(({ rarity }) => rarity === star)
				.sort(sortByName)
				.sort((a, b) => b.isNew - a.isNew)
				.sort(sortByType);
		const threeStar = list.filter(({ rarity }) => rarity === 3);
		sortedWish = [...item(5), ...item(4), ...threeStar];
	};

	$: getList(preview, previewlist);

	let container;
	let encoded;

	onMount(async () => {
		const OverlayScrollbars = (await import('overlayscrollbars')).default;
		Share = (await import('../../utility/ShareScreenshot.svelte')).default;
		playSfx = (await import('../../../helpers/audio.js')).default;

		const itemBox = container.querySelectorAll('.item-box, .shadow');
		itemBox.forEach((el) => {
			el.addEventListener('animationend', () => el.classList.remove('animate'));
		});

		OverlayScrollbars(container, {
			sizeAutoCapable: false,
			className: 'os-theme-light',
			scrollbars: { visibility: 'hidden' }
		});
		if (preview) return;
		playSfx('resultList');

		const data = sortedWish
			.map(({ name, rarity, type, isNew, fateType, stelaFortuna, outfitSet }) => {
				return `${name}/${rarity}/${type}/${+isNew}/${fateType}/${+stelaFortuna}/${+outfitSet}`;
			})
			.join('|');
		encoded = btoa(data);
	});

	const dispatch = createEventDispatcher();
	const closeHandle = () => {
		dispatch('wishEnd');
	};
</script>

<svg viewBox="0 0 302.22 1333.94" height="0" width="0" style="position: absolute;">
	<clipPath id="wishframe" transform="scale(0.003308 0.00074965)" clipPathUnits="objectBoundingBox">
		<path
			d="M0.01 168.12l0 -9.64c4.32,-21.34 12,-32.33 25.46,-25.58 -2.35,-10.3 -1.53,-26.06 5.79,-25.96 19.18,0.25 29.95,-3.14 40.24,-13.16 -4.5,-66.43 51.39,-54.26 79.61,-93.78l0 0c28.22,39.52 84.1,27.34 79.61,93.78 10.29,10.02 21.06,13.41 40.24,13.16 7.32,-0.1 8.13,15.66 5.79,25.96 13.46,-6.75 21.14,4.24 25.46,25.58l0 9.64 0.01 0 0 1004.21 -0.01 0 0 3.13c-4.32,21.34 -12,32.33 -25.46,25.58 2.35,10.3 1.53,26.06 -5.79,25.96 -19.18,-0.25 -29.95,3.14 -40.24,13.16 4.5,66.43 -51.39,54.26 -79.61,93.78l0 0c-28.22,-39.52 -84.1,-27.34 -79.61,-93.78 -10.29,-10.02 -21.06,-13.41 -40.24,-13.16 -7.32,0.1 -8.13,-15.66 -5.79,-25.96 -13.46,6.75 -21.14,-4.24 -25.46,-25.58l0 -3.13 -0.01 0 0 -1004.21 0.01 0z"
		/>
	</clipPath>
</svg>

{#if !preview}
	<button class="close" on:click={closeHandle}>
		<i class="gi-close" />
	</button>
{/if}

<div class="scroll" bind:this={container}>
	<div class="container" class:animate={!preview}>
		<div class="wishlist" bind:clientHeight style="--card-height: {clientHeight}px">
			{#each sortedWish as { name, rarity, weaponType, type, vision, wishBoxPosition, stelaFortuna, isNew, fateType, fateQty, outfitSet }, i (i)}
				<div class="item-box" class:animate={!preview} style="animation-delay: {0.5 + i * 0.1}s">
					<div
						id="wish{i}"
						class="item star{rarity} {type}"
						on:click|preventDefault={() => playSfx()}
					>
						{#if isNew}
							<div class="new">{$t('wish.result.new')}</div>
						{/if}
						<div class="item-body">
							<div class="item-content">
								<i class="gi-primo-star primo1" />
								<i class="gi-primo-star primo2" />
								<i class="gi-primo-star primo3" />
								<div class="pic">
									{#if type === 'weapon'}
										<img
											src="/images/weapons/{weaponType}/{rarity}star/{name}.webp"
											alt={name}
											class="wishpic {weaponType}-item"
											style={positionToStyle(wishBoxPosition)}
										/>
									{:else}
										<img
											src={outfitSet
												? getOutfit(name, rarity).outfitPath
												: getOutfit(name, rarity).defaultPath}
											alt={name}
											class="wishpic"
											style={positionToStyle(
												outfitSet ? getOutfit(name).wishBoxPosition : wishBoxPosition
											)}
										/>
									{/if}

									<div class="info">
										{#if type === 'weapon'}
											<img
												src="/images/utility/{weaponType}-white.svg"
												alt="{weaponType} icon"
												style="width: 60%; height: auto"
											/>
										{:else if isNew}
											<img
												src="/images/utility/icon-{vision}.svg"
												alt="Vision {vision}"
												class="{vision} filter-drop vision"
												style="height: calc(15 / 100 * var(--card-height))"
											/>
										{/if}

										{#if (isNew && type === 'character') || type === 'weapon'}
											<div class="star">
												{#each Array(rarity) as _, i}
													<div class="i gi-star" />
												{/each}
											</div>
										{/if}

										{#if type === 'character' && fateType}
											<div class="masterless {fateType}">
												<Icon type={fateType} width="80%" />
												<span>{fateQty}</span>
											</div>
										{/if}

										{#if stelaFortuna}
											<div class="stella stella{rarity}">
												<img
													src="/images/utility/stella-fortuna-{rarity}star.webp"
													alt="Stella Formula"
												/>
											</div>
										{/if}
									</div>

									{#if type === 'character' && fateType}
										<div class="cover" />
									{/if}
								</div>
							</div>
						</div>
					</div>
				</div>
			{/each}
		</div>
		<div class="shadows" style="--card-height: {clientHeight}px">
			{#each sortedWish as { rarity, type, fateType }, i}
				<div
					class="shadow shadow{rarity}"
					class:animate={!preview}
					style="animation-delay: {0.5 + i * 0.1}s"
				>
					{#if fateType && type === 'character'}
						<span class="convertion"> {$t('wish.result.convertion')} </span>
					{/if}
				</div>
			{/each}
		</div>
	</div>
</div>

{#if !preview}
	<div class="share">
		<svelte:component
			this={Share}
			encodedData={encoded}
			page="wishlist"
			item={getName(sortedWish[0].name)}
		/>
	</div>
{/if}

<style>
	.scroll {
		width: 100%;
		height: 100%;
		display: block;
	}
	.container {
		width: 100% !important;
		height: 100%;
		display: flex;
		justify-content: center;
		align-items: center;
	}
	.container.animate::after {
		content: '';
		display: block;
		width: 100%;
		height: 100%;
		position: fixed;
		top: 0;
		left: 0;
		background-color: #fff;
		opacity: 1;
		animation: reveal forwards 1s 1;
		pointer-events: none;
	}

	.close {
		position: fixed;
		top: 15px;
		right: 3%;
		z-index: 10;
		opacity: 0;
		animation: fadeIn forwards 1s 1;
	}
	:global(.mobile) .close {
		top: 0.3rem;
		right: 2%;
	}

	/* Wish List */
	.wishlist,
	.shadows {
		display: block;
		height: 55vh;
		max-height: 430px;
		min-height: 300px;
		width: 100%;
		padding: 0 20px;
		white-space: nowrap;
		text-align: center;
	}

	:global(.mobile) .wishlist,
	:global(.mobile) .shadows {
		min-height: 70vh;
	}

	:global(.mobile) .wishlist,
	:global(.mobile) .shadows {
		max-height: 72.5vh;
	}

	.item,
	.shadow {
		height: 100%;
		aspect-ratio: 1/4.5;
		border-radius: 100%;
		filter: drop-shadow(0px 0px 6px rgb(101, 187, 246));
		position: relative;
	}

	.item {
		transition: all 0.2s;
		will-change: transform;
	}
	.item:hover,
	.item.hover {
		transform: scale(1.04);
		/* z-index: +10; */
	}
	.item:hover .cover,
	.item.hover .cover {
		background-color: rgba(0, 0, 0, 0);
	}

	.shadow5,
	.shadow4,
	.item.star4,
	.item.star5 {
		filter: unset;
	}

	.item-box {
		height: 100%;
		display: inline-block;
	}
	.shadows {
		position: absolute;
		top: 50%;
		left: 50%;
		transform: translate(-50%, -50%);
		z-index: -1;
	}

	.item-box.animate,
	.shadow.animate {
		opacity: 0;
		animation: wishReveal forwards 0.4s 1;
		pointer-events: none;
	}

	.shadow {
		display: inline-block;
		height: 100%;
		border-radius: 100%;
	}
	.shadow span {
		position: absolute;
		left: 50%;
		bottom: -5%;
		color: #fff;
		transform: translateX(-50%);
		-webkit-text-stroke: 0.3px #000;
	}

	.shadow.shadow5 {
		box-shadow: 0 0 4rem rgba(255, 255, 255, 0.5), 0 0 1rem rgb(249, 170, 2),
			0 0 1.4rem rgb(249, 170, 2), 0 0 2rem rgb(249, 121, 2);
		background-color: rgb(249, 170, 2);
		filter: unset;
	}
	.star5 .item-body {
		background-image: linear-gradient(
			to bottom,
			rgb(249, 170, 2),
			rgb(255, 255, 255),
			rgb(249, 170, 2)
		);
	}

	.shadow.shadow4 {
		box-shadow: 0 0 4rem rgba(255, 255, 255, 0.6), 0 0 1rem rgb(138, 3, 161),
			0 0 1.4rem rgb(217, 0, 255), 0 0 2rem rgb(29, 4, 255);
		background-color: rgb(185, 18, 214);
	}
	.star4 .item-body {
		background-image: linear-gradient(
			to bottom,
			rgb(196, 77, 218),
			rgb(255, 255, 255),
			rgb(196, 77, 218)
		);
	}

	.new {
		position: absolute;
		z-index: 10;
		display: block;
		top: 3%;
		left: 80%;
		transform: translateX(-50%);
		padding: 0 0.4rem;
		border-radius: calc(2 / 100 * var(--card-height)) 0;
		color: #fffa66;
		background-color: #c3882a;
		border: 1px solid #fffa66;
		font-size: calc(3 / 100 * var(--card-height));
	}

	.convertion {
		font-size: calc(3 / 100 * var(--card-height));
	}

	.cover {
		display: block;
		position: absolute;
		left: 0;
		top: 0;
		height: 100%;
		width: 100%;
		transition: all 0.2s;
		background-color: rgba(0, 0, 0, 0.2);
	}

	.item-body {
		height: 100%;
		clip-path: url(#wishframe);
		margin: 0 0.5px;
		display: flex;
		justify-content: center;
		align-items: center;
		background-image: linear-gradient(to bottom, #aac8f1, #fff, #aac8f1);
	}

	.item-content {
		width: 97%;
		height: 99%;
		clip-path: url(#wishframe);
		background-image: linear-gradient(
			to bottom,
			rgb(82, 107, 129),
			rgb(187, 197, 172),
			rgb(82, 107, 129)
		);
	}

	.pic {
		width: 100%;
		height: 100%;
		position: relative;
		overflow: hidden;
	}
	.pic::after {
		content: '';
		display: block;
		position: absolute;
		bottom: 0;
		left: 0;
		width: 100%;
		height: 50%;
		background-image: linear-gradient(to top, rgba(0, 0, 0, 0.5), rgba(0, 0, 0, 0));
	}
	.pic img.wishpic {
		position: absolute;
		top: 75%;
		left: 50%;
		transform: translate(-50%, -50%);
		height: 180%;
	}

	.weapon .pic img.wishpic {
		top: 50%;
		height: 100%;
		filter: drop-shadow(0.2rem 0.2rem 0.05rem rgb(0, 0, 0));
	}
	.pic > .wishpic.catalyst-item {
		height: 35% !important;
	}

	.stella {
		margin-top: 10%;
	}
	.stella,
	.masterless {
		width: 70%;
		border-radius: 10%;
		overflow: hidden;
		display: block;
		position: relative;
		margin-left: auto;
		margin-right: auto;
		box-shadow: 0 0 7px rgba(255, 255, 255, 0.7), 0 0 14px rgba(255, 255, 255, 0.5),
			0 0 21px rgba(255, 255, 255, 0.2), 0 0 42px rgba(255, 255, 255, 1);
	}
	.stella::after,
	.masterless span {
		width: 100%;
		font-size: calc(3 / 100 * var(--card-height));
		content: '1';
		display: block;
		position: absolute;
		left: 0;
		bottom: 0;
		color: #fff;
		text-align: center;
		background-color: rgba(0, 0, 0, 0.5);
	}
	.stella img,
	.masterless img {
		width: 80%;
		height: auto;
	}

	.stella5,
	.starglitter {
		background-image: linear-gradient(to top, #ca8937, #a47853);
	}

	.stella4 {
		background-image: linear-gradient(to top, #a47ab9, #7e78a9);
	}

	.info {
		position: absolute;
		bottom: 10%;
		width: 100%;
		text-align: center;
		z-index: 10;
	}

	.info .gi-star {
		color: #f7cf33;
		display: inline-block;
		font-size: calc(3.5 / 100 * var(--card-height));
	}

	.gi-primo-star {
		color: #fff;
		position: absolute;
		left: 50%;
		transform: translateX(-50%);
	}
	.primo1 {
		top: 3%;
		font-size: calc(2 / 100 * var(--card-height));
	}
	.primo2 {
		top: 10%;
		font-size: calc(5 / 100 * var(--card-height));
	}
	.primo3 {
		top: 20%;
		font-size: calc(2 / 100 * var(--card-height));
	}

	.share {
		display: flex;
		width: 100%;
		text-align: right;
		justify-content: flex-end;
		align-items: center;
		position: absolute;
		bottom: 5%;
		right: 8%;
		color: #fff;
		font-size: 0.8rem;
		z-index: 999;
	}
	.share span {
		display: flex;
		align-items: center;
	}

	@keyframes wishReveal {
		0% {
			transform: translateX(150%);
		}
		100% {
			transform: translateX(0);
			opacity: 1;
		}
	}

	@keyframes reveal {
		20% {
			opacity: 1;
		}
		99% {
			opacity: 0;
		}
		100% {
			opacity: 0;
		}
	}

	@keyframes fadeIn {
		0% {
			opacity: 0;
		}
		100% {
			opacity: 1;
		}
	}
</style>
