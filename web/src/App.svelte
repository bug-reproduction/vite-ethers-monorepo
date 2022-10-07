<script lang="ts">
	import TwglTest from './components/TWGLTest.svelte';

	import {BigNumber} from '@ethersproject/bignumber';

	let bn = BigNumber.from(1676767);
</script>

{bn.toHexString()}

<TwglTest />
