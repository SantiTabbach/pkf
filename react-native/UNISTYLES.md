```typescript
import { PixelRatio } from 'react-native';
import type { UnistylesPlugin } from 'react-native-unistyles';

// This is my reference, take yours from your design
export const REFERENCE_WIDTH = 390;
export const REFERENCE_HEIGHT = 844;

export const calculateScaleFactor = (
	screenWidth: number,
	screenHeight: number
) => {
	const scaleFactorWidth = screenWidth / REFERENCE_WIDTH;
	const scaleFactorHeight = screenHeight / REFERENCE_HEIGHT;

	return Math.min(scaleFactorWidth, scaleFactorHeight);
};

export const autoScalePlugin: UnistylesPlugin = {
	name: 'autoScalePlugin',
	onParsedStyle: (styleKey, style, runtime) => {
		const scaleFactor = calculateScaleFactor(
			runtime.screen.width,
			runtime.screen.height
		);
		const pairs = Object.entries(style).map(([key, value]) => {
			if (styleKey.includes('unscaled')) {
				return [key, value];
			}
			const isNumber = typeof value === 'number';
			if (!isNumber || key === 'flex') {
				return [key, value];
			}

			return [key, PixelRatio.roundToNearestPixel(value * scaleFactor)];
		});

		return Object.fromEntries(pairs);
	},
};
```
