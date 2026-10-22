<script lang="ts">
    import { onMount, onDestroy } from "svelte";
    import {
        ChevronLeft,
        ChevronRight,
        BookOpen,
        Volume2,
        VolumeX,
        Sparkles,
        Layers,
        Info,
    } from "@lucide/svelte";

    interface MagazinePage {
        id: number;
        image: string;
        bgGradient: string;
    }

    const pages: MagazinePage[] = [
        {
            id: 1,
            image: "/1.png",
            bgGradient: "from-amber-950 via-stone-900 to-zinc-950",
        },
        {
            id: 2,
            image: "/2.png",
            bgGradient: "from-stone-100 to-zinc-200",
        },
        {
            id: 3,
            image: "/3.png",
            bgGradient: "from-stone-100 to-emerald-50",
        },
        {
            id: 4,
            image: "/4.png",
            bgGradient: "from-stone-100 to-orange-50",
        },
        {
            id: 5,
            image: "/5.png",
            bgGradient: "from-zinc-950 via-stone-900 to-neutral-950",
        },
    ];

    let currentIndex = $state(0);
    let isAnimating = $state(false);
    let soundEnabled = $state(false);
    let touchStartX = 0;
    let touchEndX = 0;

    let audioCtx: AudioContext | null = null;
    function playPageSound(): void {
        if (!soundEnabled) return;
        try {
            if (!audioCtx) {
                const AudioContextClass =
                    window.AudioContext ||
                    (
                        window as unknown as {
                            webkitAudioContext: typeof AudioContext;
                        }
                    ).webkitAudioContext;
                audioCtx = new AudioContextClass();
            }
            if (audioCtx.state === "suspended") {
                audioCtx.resume();
            }
            const oscillator = audioCtx.createOscillator();
            const gainNode = audioCtx.createGain();

            oscillator.type = "sine";
            oscillator.frequency.setValueAtTime(120, audioCtx.currentTime);
            oscillator.frequency.exponentialRampToValueAtTime(
                40,
                audioCtx.currentTime + 0.25,
            );

            gainNode.gain.setValueAtTime(0.15, audioCtx.currentTime);
            gainNode.gain.exponentialRampToValueAtTime(
                0.001,
                audioCtx.currentTime + 0.25,
            );

            oscillator.connect(gainNode);
            gainNode.connect(audioCtx.destination);

            oscillator.start();
            oscillator.stop(audioCtx.currentTime + 0.25);
        } catch (e) {
            console.warn("Web Audio error:", e);
        }
    }

    function goToNext(): void {
        if (isAnimating || currentIndex >= pages.length - 1) return;
        isAnimating = true;
        playPageSound();
        currentIndex += 1;
        setTimeout(() => {
            isAnimating = false;
        }, 600);
    }

    function goToPrev(): void {
        if (isAnimating || currentIndex <= 0) return;
        isAnimating = true;
        playPageSound();
        currentIndex -= 1;
        setTimeout(() => {
            isAnimating = false;
        }, 600);
    }

    function handleKeydown(event: KeyboardEvent): void {
        if (event.key === "ArrowRight" || event.key === "PageDown") {
            goToNext();
        } else if (event.key === "ArrowLeft" || event.key === "PageUp") {
            goToPrev();
        }
    }

    function handleTouchStart(e: TouchEvent): void {
        touchStartX = e.changedTouches[0].screenX;
    }

    function handleTouchEnd(e: TouchEvent): void {
        touchEndX = e.changedTouches[0].screenX;
        handleSwipe();
    }

    function handleSwipe(): void {
        const threshold = 50;
        if (touchStartX - touchEndX > threshold) {
            goToNext();
        } else if (touchEndX - touchStartX > threshold) {
            goToPrev();
        }
    }

    onMount(() => {
        window.addEventListener("keydown", handleKeydown);
    });

    onDestroy(() => {
        if (typeof window !== "undefined") {
            window.removeEventListener("keydown", handleKeydown);
        }
    });
</script>

<svelte:head>
    <link rel="preconnect" href="https://fonts.googleapis.com" />
    <link rel="preconnect" href="https://fonts.gstatic.com" />
    <link
        href="https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@300;400;500;600;700&display=swap"
        rel="stylesheet"
    />
</svelte:head>

<!-- svelte-ignore a11y_no_static_element_interactions -->
<div
    class="min-h-screen bg-stone-950 text-stone-100 flex flex-col justify-between select-none overflow-x-hidden font-sans relative"
    ontouchstart={handleTouchStart}
    ontouchend={handleTouchEnd}
>
    <!-- Ambient Background Glows -->
    <div
        class="absolute top-0 left-1/4 w-96 h-96 bg-amber-600/10 rounded-full blur-3xl pointer-events-none"
    ></div>
    <div
        class="absolute bottom-0 right-1/4 w-96 h-96 bg-blue-600/10 rounded-full blur-3xl pointer-events-none"
    ></div>

    <!-- Top Header Navbar -->
    <header
        class="w-full px-6 py-4 flex items-center justify-between border-b border-stone-800/60 backdrop-blur-md bg-stone-950/80 z-20"
    >
        <div class="flex items-center gap-3">
            <div
                class="p-2 rounded-xl bg-amber-500/10 border border-amber-500/20 text-amber-400"
            >
                <BookOpen class="w-5 h-5" />
            </div>
            <div>
                <h1
                    class="text-sm font-bold tracking-widest text-stone-200 uppercase"
                >
                    SIDE B
                </h1>
                <p class="text-xs text-stone-400">Revista Virtual</p>
            </div>
        </div>

        <div
            class="hidden md:flex items-center gap-2 px-3 py-1.5 rounded-full bg-stone-900/80 border border-stone-800 text-xs text-stone-300"
        >
            <Sparkles class="w-3.5 h-3.5 text-amber-400" />
            <span>Usa <b>← / →</b> flechas de dirección o swipe izquierda / derecha</span>
        </div>

        <div class="flex items-center gap-2">
            <button
                onclick={() => (soundEnabled = !soundEnabled)}
                class="p-2.5 rounded-xl bg-stone-900 border border-stone-800 hover:bg-stone-800 text-stone-300 transition-all flex items-center gap-2 text-xs font-medium cursor-pointer shadow-sm active:scale-95"
                title="Toggle Page Turn Sound"
            >
                {#if soundEnabled}
                    <Volume2 class="w-4 h-4 text-amber-400" />
                    <span class="hidden sm:inline">Sound On</span>
                {:else}
                    <VolumeX class="w-4 h-4 text-stone-500" />
                    <span class="hidden sm:inline">Sound Off</span>
                {/if}
            </button>
        </div>
    </header>

    <!-- Main Stage / Magazine Viewer -->
    <main
        class="flex-1 flex flex-col items-center justify-center p-4 sm:p-8 perspective-1000 my-auto"
    >
        <div
            class="relative w-full max-w-5xl aspect-[4/3] sm:aspect-[16/10] flex items-center justify-center"
        >
            <!-- Shadow under book -->
            <div
                class="absolute -bottom-6 w-11/12 h-12 bg-black/70 blur-xl rounded-full pointer-events-none"
            ></div>

            <!-- The 3D Book Object -->
            <div
                class="relative w-full h-full bg-stone-900 rounded-2xl shadow-2xl border border-stone-800/80 flex overflow-hidden transform-style-3d ring-1 ring-white/10"
            >
                <!-- Spine shadow overlay in center -->
                <div
                    class="absolute left-1/2 top-0 bottom-0 w-12 -ml-6 bg-gradient-to-r from-transparent via-black/40 to-transparent pointer-events-none z-10"
                ></div>

                <!-- Render All Pages stacked with absolute positioning and CSS 3D transitions -->
                {#each pages as page, idx}
                    {@const isPast = idx < currentIndex}
                    {@const isCurrent = idx === currentIndex}

                    <div
                        class="absolute inset-0 w-full h-full transition-all duration-700 ease-in-out transform-style-3d origin-left bg-gradient-to-br {page.bgGradient}"
                        style="
                            transform: rotateY({isPast ? '-180deg' : '0deg'});
                            z-index: {isCurrent ? 20 : pages.length - idx};
                            pointer-events: {isCurrent ? 'auto' : 'none'};
                            box-shadow: {isPast
                            ? 'inset -10px 0 20px rgba(0,0,0,0.5)'
                            : 'none'};
                        "
                    >
                        <!-- Realistic Page Turn Shading Gradient -->
                        <div
                            class="absolute inset-0 pointer-events-none transition-opacity duration-500 z-30"
                            style="background: {isPast
                                ? 'linear-gradient(to right, rgba(0,0,0,0.4) 0%, rgba(0,0,0,0) 15%)'
                                : isCurrent
                                  ? 'linear-gradient(to left, rgba(0,0,0,0.2) 0%, rgba(0,0,0,0) 10%)'
                                  : 'none'};"
                        ></div>

                        <!-- Side-by-Side Images Layout for Tall Images -->
                        <div
                            class="w-full h-full grid grid-cols-1 md:grid-cols-2 p-4 sm:p-8 gap-4 sm:gap-6 items-center justify-center overflow-hidden"
                        >
                            <!-- Left Image -->
                            <div
                                class="w-full h-full flex flex-col justify-center items-center relative group min-h-0"
                            >
                                <div
                                    class="absolute -inset-2 bg-gradient-to-r from-amber-500/10 to-transparent rounded-2xl blur-lg opacity-50 group-hover:opacity-100 transition duration-500"
                                ></div>
                                <div
                                    class="relative rounded-xl overflow-hidden shadow-2xl border border-white/10 w-full h-full flex items-center justify-center bg-stone-900/50 p-2 min-h-0"
                                >
                                    <img
                                        src={page.image}
                                        alt="Magazine Plate Left"
                                        class="w-full h-full object-contain transform hover:scale-[1.02] transition duration-700"
                                    />
                                    <div
                                        class="absolute bottom-3 left-3 px-3 py-1 rounded-full bg-black/60 backdrop-blur-md text-[10px] tracking-widest uppercase text-stone-300 border border-white/10 z-10"
                                    >
                                        Plana {idx * 2 + 1}
                                    </div>
                                </div>
                            </div>

                            <!-- Right Image -->
                            <div
                                class="w-full h-full flex flex-col justify-center items-center relative group min-h-0"
                            >
                                <div
                                    class="absolute -inset-2 bg-gradient-to-r from-transparent to-amber-500/10 rounded-2xl blur-lg opacity-50 group-hover:opacity-100 transition duration-500"
                                ></div>
                                <div
                                    class="relative rounded-xl overflow-hidden shadow-2xl border border-white/10 w-full h-full flex items-center justify-center bg-stone-900/50 p-2 min-h-0"
                                >
                                    <img
                                        src={pages[(idx + 1) % pages.length]
                                            .image}
                                        alt="Magazine Plate Right"
                                        class="w-full h-full object-contain transform hover:scale-[1.02] transition duration-700"
                                    />
                                    <div
                                        class="absolute bottom-3 left-3 px-3 py-1 rounded-full bg-black/60 backdrop-blur-md text-[10px] tracking-widest uppercase text-stone-300 border border-white/10 z-10"
                                    >
                                        Plana {idx * 2 + 2}
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                {/each}
            </div>
        </div>
    </main>

    <!-- Bottom Navigation & Controls Footer -->
    <footer
        class="w-full px-6 py-5 flex flex-col sm:flex-row items-center justify-between gap-4 border-t border-stone-800/60 backdrop-blur-md bg-stone-950/80 z-20"
    >
        <div class="flex items-center gap-2 text-xs text-stone-400">
            <Layers class="w-4 h-4 text-amber-400" />
            <span
                >Página <strong class="text-stone-200"
                    >{currentIndex + 1}</strong
                >
                de <strong class="text-stone-200">{pages.length}</strong></span
            >
        </div>

        <!-- Page Navigation Buttons -->
        <div class="flex items-center gap-3">
            <button
                onclick={goToPrev}
                disabled={currentIndex === 0 || isAnimating}
                class="px-4 py-2.5 rounded-xl bg-stone-900 border border-stone-800 hover:bg-stone-800 disabled:opacity-40 disabled:cursor-not-allowed text-stone-200 text-xs font-medium transition-all flex items-center gap-2 cursor-pointer shadow-md active:scale-95"
            >
                <ChevronLeft class="w-4 h-4" />
                <span>Página Previa</span>
            </button>

            <div class="flex gap-1.5 px-2">
                {#each pages as _, pIdx}
                    <button
                        onclick={() => {
                            if (!isAnimating) {
                                playPageSound();
                                currentIndex = pIdx;
                            }
                        }}
                        class="w-2.5 h-2.5 rounded-full transition-all cursor-pointer {pIdx ===
                        currentIndex
                            ? 'bg-amber-400 w-6'
                            : 'bg-stone-800 hover:bg-stone-700'}"
                        title="Go to spread {pIdx + 1}"
                    ></button>
                {/each}
            </div>

            <button
                onclick={goToNext}
                disabled={currentIndex === pages.length - 1 || isAnimating}
                class="px-4 py-2.5 rounded-xl bg-amber-500 hover:bg-amber-400 text-stone-950 disabled:opacity-40 disabled:cursor-not-allowed text-xs font-semibold transition-all flex items-center gap-2 cursor-pointer shadow-lg shadow-amber-500/10 active:scale-95"
            >
                <span>Siguiente Página</span>
                <ChevronRight class="w-4 h-4" />
            </button>
        </div>

        <div class="flex items-center gap-2 text-xs text-stone-500">
            <Info class="w-3.5 h-3.5" />
            <span>Pure Visual 3D Página</span>
        </div>
    </footer>
</div>

<style>
    .perspective-1000 {
        perspective: 1500px;
    }

    .transform-style-3d {
        transform-style: preserve-3d;
    }

    body {
        font-family: "Plus Jakarta Sans", sans-serif;
    }
</style>
