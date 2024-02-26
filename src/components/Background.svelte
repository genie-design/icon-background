<script context="module" lang="ts">
	import JSZip from 'jszip';
	import { Box, System } from 'detect-collisions';

	export type BackgroundProps = {
		xMax?: number;
		yMax?: number;
		xStart?: number;
		yStart?: number;
		rotations?: string[];
		icons?: string[];
		textColors?: string[];
		padding?: number;
		maxWidth?: 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10;
		blank?: {
			left: number;
			top: number;
			width: number;
			height: number;
			circle?: boolean;
			ellipse?: boolean;
			padding?: number;
		};
		flippableIcons?: string[];
		rotateableIcons?: string[];
	};

	export type Setting = string | 'inherit';
	export type ScreenSettings = {
		xs?: Setting;
		sm?: Setting;
		md?: Setting;
		lg?: Setting;
		xl?: Setting;
		twoxl?: Setting;
	};
	export type Icon =
		| 'paw-prints'
		| 'heart-doodle'
		| 'heart'
		| 'paw-two-tone'
		| 'paw-print'
		| 'paw-fill'
		| 'heart-fill'
		| 'heart-suit'
		| 'heart-two-tone';

	export type BGIcon = {
		name: Icon;
		color: string;
		size?: string | ScreenSettings;
		style: { top: string; left: string } & Partial<HTMLDivElement['style']>;
		classes?: string;
	};
	export type AddedIcon = {
		left: number;
		top: number;
		widthAndHeight: number;
		color: string;
		icon: Icon;
		flipped?: boolean;
		rotateClass?: string;
	};
</script>

<script lang="ts">
	import BackgroundIcon from './BackgroundIcon.svelte';
	export let xMax: BackgroundProps['xMax'] = 0;
	export let yMax: BackgroundProps['yMax'] = 0;
	export let maxWidth: BackgroundProps['maxWidth'] = 9;
	const maxWidthSetting = maxWidth && maxWidth < 10 ? maxWidth : 9;
	export let padding: BackgroundProps['padding'] = undefined;
	export let blank: BackgroundProps['blank'] = undefined;
	export let unzippedIconConfigs: AddedIcon[] | undefined = undefined;
	const generateIconPositions = (options?: {
		xStart?: number;
		yStart?: number;
		xMax?: number;
		yMax?: number;
		padding?: number;
	}) => {
		options = options || {};
		const { xStart, yStart, padding } = options;
		let xMax = options.xMax || 0;
		let yMax = options.yMax || 0;
		const icons: Icon[] = [
			'paw-prints',
			'heart-doodle',
			'heart',
			'paw-two-tone',
			'paw-print',
			'paw-fill',
			'heart-fill',
			'heart-suit',
			'heart-two-tone'
		] as const;

		const addedIcons: AddedIcon[] = [];
		const iconsCount: Record<string, number> = {};
		icons.forEach((icon) => {
			iconsCount[icon] = 0;
		});
		const colorsCount: Record<string, number> = {
			'text-base-300': 0,
			'text-primary': 0,
			'text-secondary': 0,
			'text-accent': 0,
			'text-info': 0
		};
		const rotateClasses = [
			'rotate-6',
			'rotate-12',
			'rotate-45',
			'-rotate-6',
			'-rotate-12',
			'-rotate-45'
		];
		const flipIcons = ['paw-prints', 'heart-fill'];

		const rotateIcons = ['paw-print', 'paw-fill', 'paw-two-tone'];
		const min = 32;
		const maxMax = maxWidthSetting * 16;
		const getWidth = (currMax: number) => {
			const max = Math.min(maxMax, currMax);
			return Math.max(min, Math.floor(Math.random() * ((max - min) / 8) + min / 8) * 8);
		};
		let y = yStart || 0;
		const system = new System();
		if (blank) {
			if (blank.circle) {
				system.createCircle(
					{
						x: blank.left + blank.width / 2,
						y: blank.top + blank.height / 2
					},
					blank.width / 2,
					{
						padding: blank.padding
					}
				);
			} else if (blank.ellipse) {
				const radiusX = blank.width / 2;
				const radiusY = blank.height / 2;

				system.createEllipse(
					{
						x: blank.left + blank.width / 2,
						y: blank.top + blank.height / 2
					},
					radiusX,
					radiusY,
					(radiusX + radiusY) / Math.PI,
					{
						padding: blank.padding
					}
				);
			} else {
				system.createBox(
					{
						x: blank.left,
						y: blank.top
					},
					blank.width,
					blank.height,
					{
						padding: blank.padding
					}
				);
			}
		}
		while (y < yMax) {
			let x = xStart || 0;
			while (x < xMax) {
				const previousIcon = addedIcons.length > 0 ? addedIcons[addedIcons.length - 1] : null;
				const prevWidthAndHeight = previousIcon ? previousIcon.widthAndHeight * 16 : 32;

				let width = min;
				let left = 0;
				let top = 0;
				let collides = true;

				let box: Box | undefined = undefined;
				let randomOffset = 0.25;
				let widthOffset = 0;
				while (collides && widthOffset <= 256 && width > 0) {
					const paddingX = prevWidthAndHeight * (Math.random() * 0.75 + randomOffset);
					const paddingY = prevWidthAndHeight * (Math.random() * 0.75 + randomOffset);
					width = getWidth(xMax - x - paddingX - widthOffset);
					top = y + paddingY;
					left = x + paddingX;
					if (left + width < xMax) {
						box = new Box(
							{
								x: left,
								y: top
							},
							width,
							width,
							{
								padding: 32
							}
						);
						system.insert(box);
						collides = system.checkOne(box, () => true);
						if (collides) {
							system.remove(box);
						}
					}
					if (widthOffset >= 128) {
						randomOffset += 0.25;
					}
					widthOffset += 16;
				}

				if (box && !collides) {
					const leastColor = Math.min(...Object.values(colorsCount)) + 1;
					const colorsWithLeast = Object.keys(colorsCount).filter(
						(color) => colorsCount[color] <= leastColor
					);
					const color = colorsWithLeast[Math.floor(Math.random() * colorsWithLeast.length)];
					colorsCount[color]++;
					const leastIcon = Math.min(...Object.values(iconsCount)) + 1;
					const iconsWithLeast = icons.filter((icon) => iconsCount[icon] <= leastIcon);
					const icon = iconsWithLeast[Math.floor(Math.random() * iconsWithLeast.length)];
					let flipped = false;
					if (flipIcons.includes(icon) && Math.random() > 0.5) {
						flipped = true;
					}
					let rotateClass = '';
					if (rotateIcons.includes(icon) && Math.random() > 0.2) {
						rotateClass += rotateClasses[Math.floor(Math.random() * rotateClasses.length)];
					}
					iconsCount[icon]++;
					// system.insert(box);
					const iconToAdd = {
						left,
						top,
						widthAndHeight: width / 16,
						color,
						icon,
						flipped,
						rotateClass
					};
					addedIcons.push(iconToAdd);
					x = left + width;
				} else {
					x = x + width;
				}
			}
			y += maxWidthSetting * 16;
		}

		return addedIcons;
	};
	let background: BGIcon[] = [];
	let iconConfigs: AddedIcon[] = unzippedIconConfigs || [];
	if (iconConfigs.length === 0) {
		iconConfigs = generateIconPositions({
			xStart: 0,
			yStart: 0,
			xMax,
			yMax,
			padding: padding ?? 16
		});
	}

	const getBackground = () => {
		try {
			const rect = {
				left: 0,
				top: 0,
				width: 0,
				height: 0,
				circle: false,
				...blank
			};

			background = iconConfigs.reduce<BGIcon[]>((acc, pos) => {
				let classes = '';
				const icon = pos.icon;
				if (pos.flipped) {
					classes += ' scale-x-[-1]';
				}
				if (pos.rotateClass) {
					classes += ' ' + pos.rotateClass;
				}
				const bgIcon: BGIcon = {
					name: icon,
					color: `${pos.color}`,
					size: `text-[${pos.widthAndHeight}rem]`,
					style: {
						top: `${pos.top}px`,
						left: `${pos.left}px`
					},
					classes
				};
				if (pos.top === 8) {
				}
				acc.push(bgIcon);
				return acc;
			}, []);
		} catch (e) {
			console.error(e);
		}
		return background;
	};
	$: background = iconConfigs.length ? getBackground() : [];
	const zipConfigs = async () => {
		if (iconConfigs.length > 0) {
			const zip = new JSZip();
			console.log('-----------------------');
			const configString = JSON.stringify(iconConfigs);
			zip.file('minified_configs.json', configString);
			const zipped = await zip.generateAsync({
				type: 'base64',
				compression: 'DEFLATE',
				compressionOptions: { level: 9 }
			});
			// const original = await (await zip.loadAsync(zipped, { base64: true }))
			// 	.file('minified_configs.json')
			// 	?.async('string');
			const urlEncoded = encodeURIComponent(zipped);
			// console.log(zipped);
			// console.log('0-0');
			console.log(urlEncoded);
		}
	};
	$: zipConfigs();
</script>

{#if background}
	{#each background as bg}
		<BackgroundIcon {bg} />
	{/each}
{/if}
