<script>
	import {
		Header,
		HeaderNav,
		HeaderNavItem,
		HeaderNavMenu,
		SkipToContent,
		Content,
		Grid,
		Row,
		Column,
	} from 'carbon-components-svelte';

	let isSideNavOpen = false;
</script>

<svelte:head>
	<!-- <link rel="stylesheet" href="https://unpkg.com/carbon-components-svelte/css/white.css" /> -->
	<title>MoonMeter</title>
</svelte:head>

<Header company="MoonMetrics" bind:isSideNavOpen>
	<svelte:fragment slot="skip-to-content">
		<SkipToContent />
	</svelte:fragment>
	<HeaderNav>
		<HeaderNavItem href="/about" text="About" />
		<HeaderNavItem href="/dashboard" text="Dashboard" />
		<!-- <HeaderNavMenu text="Tools">
			<HeaderNavItem href="/dash" text="Dashboard" />
			<HeaderNavItem href="/" text="ArchiveDB" />
			<HeaderNavItem href="/" text="DocScan" />
			<HeaderNavItem href="/" text="PMAT" />
		</HeaderNavMenu> -->
	</HeaderNav>
</Header>
<Content>
	<Grid>
		<Row>
			<Column>
				<slot />
			</Column>
		</Row>
	</Grid>
</Content>
