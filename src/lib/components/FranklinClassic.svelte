<script lang="ts">
	import type { Timeframe, PlannerSettings } from '$lib';
	import Grid from './Grid.svelte';

	let {
		timeframe = {} as Timeframe,
		settings = {} as PlannerSettings,
		mode = 'left' as 'left' | 'right',
	} = $props();

	// Helper to get mini calendar data
	function getMiniMonth(date: Date) {
		const year = date.getUTCFullYear();
		const month = date.getUTCMonth();
		const firstDay = new Date(Date.UTC(year, month, 1));
		const lastDay = new Date(Date.UTC(year, month + 1, 0));
		const startWeekOnSunday = settings.date.startWeekOnSunday;
		const numDaysBeforeStart =
			(firstDay.getUTCDay() + 7 - (startWeekOnSunday ? 0 : 1)) % 7;

		const days = [];
		for (let i = 0; i < numDaysBeforeStart; i++) {
			days.push(null);
		}
		for (let i = 1; i <= lastDay.getUTCDate(); i++) {
			days.push(i);
		}
		return {
			days,
			monthName: firstDay.toLocaleString('default', { month: 'short', timeZone: 'UTC' }),
			year,
			month: month + 1,
		};
	}

	const currentMonth = $derived(getMiniMonth(timeframe.start));
	const prevMonth = $derived(
		getMiniMonth(
			new Date(
				Date.UTC(timeframe.start.getUTCFullYear(), timeframe.start.getUTCMonth() - 1, 1),
			),
		),
	);
	const nextMonth = $derived(
		getMiniMonth(
			new Date(
				Date.UTC(timeframe.start.getUTCFullYear(), timeframe.start.getUTCMonth() + 1, 1),
			),
		),
	);

	const hours = $derived.by(() => {
		const start = settings.dayPage.classicStartHour ?? 8;
		const end = settings.dayPage.classicEndHour ?? 20;
		const h = [];
		for (let i = start; i <= end; i++) {
			h.push(i > 12 ? i - 12 : i === 0 ? 12 : i);
		}
		return h;
	});

	// Check if a link should be active (i.e. if the page exists)
	function hasMonthPage(year: number, month: number) {
		return settings.months.some((m) => m.year === year && m.month === month);
	}

	function hasDayPage(year: number, month: number, day: number) {
		return settings.days.some(
			(d) => d.year === year && d.month === month && d.daySinceMonth === day,
		);
	}
</script>

{#snippet LeftPage()}
	<div class="left-page">
		<header class="franklin-header">
			<div class="date-big">
				<div class="day-num">{timeframe.start.getUTCDate()}</div>
				<div class="day-details">
					<div class="weekday">
						{timeframe.start.toLocaleString('default', {
							weekday: 'long',
							timeZone: 'UTC',
						})}
					</div>
					<div class="month-year">
						{timeframe.start.toLocaleString('default', {
							month: 'long',
							year: 'numeric',
							timeZone: 'UTC',
						})}
					</div>
				</div>
			</div>

			<div class="current-month-large">
				<div class="month-label">{currentMonth.monthName} {currentMonth.year}</div>
				<div class="days-grid-large">
					{#each settings.date.startWeekOnSunday ? ['S', 'M', 'T', 'W', 'T', 'F', 'S'] : ['M', 'T', 'W', 'T', 'F', 'S', 'S'] as d}
						<div class="dow">{d}</div>
					{/each}
					{#each currentMonth.days as day}
						{#if day}
							{@const isToday = day === timeframe.start.getUTCDate()}
							{#if hasDayPage(currentMonth.year, currentMonth.month, day)}
								<a
									href="#{currentMonth.year}-{currentMonth.month}-{day}"
									class="day-cell"
									class:active={isToday}>
									{day}
								</a>
							{:else}
								<div class="day-cell" class:active={isToday}>{day}</div>
							{/if}
						{:else}
							<div class="day-cell empty"></div>
						{/if}
					{/each}
				</div>
			</div>

			<div class="mini-side-col">
				<div class="mini-month small">
					{#if hasMonthPage(prevMonth.year, prevMonth.month)}
						<a href="#{prevMonth.year}-{prevMonth.month}">
							<div class="month-name">
								{prevMonth.monthName}
								{prevMonth.year}
							</div>
							<div class="days-grid">
								{#each prevMonth.days as day}
									<div class="day-cell">{day || ''}</div>
								{/each}
							</div>
						</a>
					{:else}
						<div class="month-name">
							{prevMonth.monthName}
							{prevMonth.year}
						</div>
						<div class="days-grid">
							{#each prevMonth.days as day}
								<div class="day-cell">{day || ''}</div>
							{/each}
						</div>
					{/if}
				</div>
				<div class="mini-month small">
					{#if hasMonthPage(nextMonth.year, nextMonth.month)}
						<a href="#{nextMonth.year}-{nextMonth.month}">
							<div class="month-name">
								{nextMonth.monthName}
								{nextMonth.year}
							</div>
							<div class="days-grid">
								{#each nextMonth.days as day}
									<div class="day-cell">{day || ''}</div>
								{/each}
							</div>
						</a>
					{:else}
						<div class="month-name">
							{nextMonth.monthName}
							{nextMonth.year}
						</div>
						<div class="days-grid">
							{#each nextMonth.days as day}
								<div class="day-cell">{day || ''}</div>
							{/each}
						</div>
					{/if}
				</div>
			</div>
		</header>

		<div class="main-content">
			<div class="tasks-col">
				<h3>Prioritized Daily Task List</h3>
				<div class="task-table">
					<div class="task-header">
						<span class="task-col-extra heavy-arrow">⬇</span>
						<span class="task-col-extra">ABC</span>
						<span class="task-title">Tasks</span>
					</div>
					{#each new Array(23) as _}
						<div class="task-row">
							<div class="task-col-extra"></div>
							<div class="task-col-extra"></div>
							<div class="task-input"></div>
						</div>
					{/each}
				</div>
				<div class="daily-tracker">
					<h3>Daily Tracker</h3>
					<div class="tracker-box">
						<div class="tracker-lines">
							{#each new Array(8) as _}
								<div class="line"></div>
							{/each}
						</div>
					</div>
				</div>
			</div>
			<div class="schedule-col">
				<h3>Appointment Schedule</h3>
				<div class="schedule-table">
					{#each hours as hour, i}
						<div class="schedule-row">
							<div class="time-label">{hour}</div>
							<div class="schedule-inputs">
								<div class="half-hour"></div>
								<div class="half-hour"></div>
							</div>
						</div>
					{/each}
					<div class="schedule-row footer"></div>
				</div>
			</div>
		</div>
	</div>
{/snippet}

{#snippet RightPage()}
	<div class="right-page">
		<header>
			<div class="date-right">
				<div class="day-num">{timeframe.start.getUTCDate()}</div>
				<div class="date-info">
					<div class="weekday">
						{timeframe.start.toLocaleString('default', {
							weekday: 'long',
							timeZone: 'UTC',
						})}
					</div>
					<div class="month-year">
						{timeframe.start.toLocaleString('default', {
							month: 'long',
							year: 'numeric',
							timeZone: 'UTC',
						})}
					</div>
				</div>
			</div>
		</header>
		<div class="stats-row">
			<span>{timeframe.daySinceYear}th Day</span>
			<span>{365 - (timeframe.daySinceYear || 0)} Left</span>
			<span>Week {timeframe.weekSinceYear}</span>
		</div>

		<div class="notes-container">
			<h3>Daily Notes</h3>
			<div class="notes-grid">
				<Grid display="lined" lines={45} />
			</div>
		</div>
	</div>
{/snippet}

<div class="franklin-container {mode}">
	{#if mode === 'left'}
		{@render LeftPage()}
	{:else if mode === 'right'}
		{@render RightPage()}
	{/if}
</div>

<style lang="scss">
	.franklin-container {
		width: 100%;
		height: 100%;
		color: var(--text);
		font-family: var(--font);
		display: flex;
		flex-direction: column;
		padding: 0.5rem;
		box-sizing: border-box;

		h3 {
			font-family: var(--font-display);
			font-size: 0.9rem;
			margin: 0;
			padding: 0.2rem 0.5rem;
			background: var(--text);
			color: #fff;
			text-align: center;
			text-transform: uppercase;
			letter-spacing: 0.05em;
		}
	}

	.left-page,
	.right-page {
		display: flex;
		flex-direction: column;
		height: 100%;
	}

	.franklin-header {
		display: flex;
		justify-content: space-between;
		align-items: flex-start;
		margin-bottom: 0.5rem;
		gap: 1.5rem;
	}

	.date-big {
		display: flex;
		align-items: center;
		gap: 0.5rem;
		flex-shrink: 0;
		.day-num {
			font-size: 4rem;
			font-weight: bold;
			font-family: var(--font-display);
			line-height: 1;
		}
		.day-details {
			display: flex;
			flex-direction: column;
			.weekday {
				font-weight: bold;
				font-size: 1.4rem;
			}
			.month-year {
				font-size: 1.1rem;
				opacity: 0.8;
			}
		}
	}

	.current-month-large {
		flex: 1;
		display: flex;
		flex-direction: column;
		align-items: center;

		.month-label {
			font-size: 1.2rem;
			font-weight: bold;
			margin-bottom: 0.25rem;
			font-family: var(--font-display);
		}

		.days-grid-large {
			width: 100%;
			max-width: 400px;
			display: grid;
			grid-template-columns: repeat(7, 1fr);
			text-align: center;
			font-size: 0.75rem;
			border-top: 1px solid var(--outline);

			.dow {
				font-weight: bold;
				padding: 4px 0;
				color: var(--text);
				font-size: 0.7rem;
				border-bottom: 1px solid var(--outline);
				margin-bottom: 2px;
			}

			.day-cell {
				padding: 4px 0;
				color: inherit;
				text-decoration: none;
				display: block;
				font-size: 0.85rem;

				&.active {
					background: var(--text);
					color: #fff;
					border-radius: 2px;
				}
				&.empty {
					pointer-events: none;
				}
			}
		}
	}

	.mini-side-col {
		width: 120px;
		display: flex;
		flex-direction: column;
		gap: 0.25rem;
		flex-shrink: 0;
	}

	.mini-month {
		a {
			text-decoration: none;
			color: inherit;
			display: block;
		}
		.month-name {
			font-size: 0.6rem;
			text-align: center;
			font-weight: bold;
			margin-bottom: 2px;
		}
		.days-grid {
			display: grid;
			grid-template-columns: repeat(7, 1fr);
			font-size: 0.5rem;
			text-align: center;
			.day-cell {
				padding: 1px 0;
			}
		}
	}

	.main-content {
		display: flex;
		flex: 1;
		gap: 1rem;
		overflow: hidden;
	}

	.tasks-col {
		width: 58%;
		display: flex;
		flex-direction: column;
		gap: 0.5rem;
	}

	.task-table {
		flex: 1;
		border: 1px solid var(--text);
		display: flex;
		flex-direction: column;
		.task-header {
			display: flex;
			font-size: 0.6rem;
			font-weight: bold;
			border-bottom: 1px solid var(--text);
			.task-col-extra {
				width: 35px;
				border-right: 1px solid var(--text);
				flex-shrink: 0;
				display: flex;
				align-items: center;
				justify-content: center;

				&.heavy-arrow {
					font-size: 0.8rem;
					font-weight: 900;
				}
			}
			.task-title {
				flex: 1;
				text-align: center;
			}
		}
		.task-row {
			display: flex;
			flex: 1;
			border-bottom: 1px solid var(--outline);
			&:last-child {
				border-bottom: none;
			}
			.task-col-extra {
				width: 35px;
				border-right: 1px solid var(--text);
				flex-shrink: 0;
			}
			.task-input {
				flex: 1;
			}
		}
	}

	.daily-tracker {
		.tracker-box {
			border: 1px solid var(--text);
			padding: 0.2rem;
			.tracker-lines {
				.line {
					height: 0.8rem;
					border-bottom: 1px solid var(--outline);
					&:last-child {
						border-bottom: none;
					}
				}
			}
		}
	}

	.schedule-col {
		width: 42%;
		display: flex;
		flex-direction: column;
	}

	.schedule-table {
		flex: 1;
		border: 1px solid var(--text);
		display: flex;
		flex-direction: column;
		.schedule-row {
			display: flex;
			flex: 1;
			border-bottom: 1px solid var(--text);
			&:last-child {
				border-bottom: none;
			}
			&.footer {
				height: 1.5rem;
				flex: none;
			}
			.time-label {
				width: 30px;
				font-size: 0.8rem;
				font-weight: bold;
				display: flex;
				align-items: center;
				justify-content: center;
				border-right: 1px solid var(--text);
			}
			.schedule-inputs {
				flex: 1;
				display: flex;
				flex-direction: column;
				.half-hour {
					flex: 1;
					&:first-child {
						border-bottom: 1px solid var(--outline);
					}
				}
			}
		}
	}

	.right-page {
		display: flex;
		flex-direction: column;
		height: 100%;
		header {
			justify-content: flex-end;
			align-items: center;
		}
		.date-right {
			display: flex;
			align-items: center;
			gap: 0.5rem;
			text-align: right;
			.day-num {
				font-size: 4rem;
				font-weight: bold;
				font-family: var(--font-display);
			}
			.date-info {
				.weekday {
					font-weight: bold;
					font-size: 1.4rem;
				}
				.month-year {
					font-size: 1.1rem;
					opacity: 0.8;
				}
			}
		}
		.stats-row {
			display: flex;
			justify-content: flex-end;
			gap: 1rem;
			font-size: 0.6rem;
			margin-bottom: 0.5rem;
			border-bottom: 2px solid var(--text);
			padding-bottom: 2px;
		}
		.notes-container {
			flex: 1;
			display: flex;
			flex-direction: column;
			.notes-grid {
				flex: 1;
				border: 1px solid var(--text);
				margin-top: -1px;
			}
		}
	}
</style>
