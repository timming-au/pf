<svelte:head>
	<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.9.1/gsap.min.js"></script>
	<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.9.1/ScrollTrigger.min.js"></script>
	<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.9.1/ScrollToPlugin.min.js"></script>
	<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.9.1/MotionPathPlugin.min.js"></script>
	<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.9.1/TextPlugin.min.js"></script>
</svelte:head>
<svelte:window bind:innerHeight bind:innerWidth></svelte:window>
<script lang="ts" context="module">
    declare const gsap;
</script>
<script lang="ts">
    import { prefetch } from "$app/navigation";
    import { onDestroy, onMount } from "svelte";
    let innerWidth, innerHeight, timeout
    let hexagonPathNode, hexagonNode, textNode, playNode, playCircleNode, playArrowNode, textContainerNode
    onMount(() => {
        // window
        innerWidth = window.innerWidth
        innerHeight = window.innerHeight
        timeout = []
        // DOM Nodes
        hexagonNode = document.getElementsByClassName("hexagon")
        hexagonPathNode = document.getElementsByClassName("hexagon-path")
        textNode = document.getElementById("text")
        textContainerNode = document.getElementById("text-c")
        playNode = document.getElementById("play")
        playCircleNode = document.getElementById("play-circle-path")
        playArrowNode = document.getElementById("play-arrow")

        /// init variables
        let initTimeline = gsap.timeline()
        let initDrawSvg = [];
        let initMoveSvg = [];

        /// text variables
        let sentences = ["Preparing ingredients...", "Sketching webpages...", "Plating elements...", "Sprinkling fine touches...", "Pestling up hues..."]
        let sentencesLen = sentences.length
        let textTimeline = gsap.timeline({ defaults:{ease:"power1.inOut",}, })
        let iterator = 0;

        /// prefetch variables
        let routes = ["/about", "/contact", "/project"]
        let isLoaded = false
        let isLoadedCount = 0

        /// play variables


        /// init animations
        initTimeline.set(hexagonPathNode, {
            strokeDasharray: i => hexagonPathNode[i].getTotalLength(),
        })
        initTimeline.set(hexagonNode,{
            opacity: 1,
            x: i => (routes.length * 28 / (routes.length - 1) * i) - routes.length * 28 / 2,
        })
        // hexagonNode.length should match routes.length
        for(let i = 0, n = hexagonNode.length; i < n; i++){
            initDrawSvg[i] = gsap.from(hexagonPathNode[i],{
                id: `${routes[i]}initDrawSvg`,
                duration: 0.7,
                delay: i * 0.12,
                strokeDashoffset: hexagonPathNode[i].getTotalLength(),
            })
            initMoveSvg[i] = gsap.to(hexagonNode[i], {
                id: `${routes[i]}initMoveSvg`,
                duration: 1.5,
                delay: i * 0.3,
                y: 20,
                repeat: -1,
                yoyo: true,
                ease: "power1.inOut",
            })
        }
        animText()
        function animText(){
            if(!isLoaded){
                if(textTimeline.iteration() == 1) textTimeline.restart()
                /// text animations
                if (iterator > sentencesLen - 1){
                    iterator = 0
                }
                gsap.set(textNode,{
                    innerHTML:sentences[iterator]
                })
                iterator++
                textTimeline.to(textNode,{
                    duration: 0.4,
                    opacity: 1,
                    ease: "power1.out",
                    y: "0",
                })
                textTimeline.to(textNode,{
                    duration: 0.4,
                    delay: 2,
                    opacity: 0,
                    ease: "power1.out",
                    y: "1.2rem",
                    onComplete:animText
                })
            }else{
                textTimeline.kill()
                let finishTimeline = gsap.timeline()
                finishTimeline.to(textNode,{
                    duration: 0.4,
                    opacity: 0,
                    ease: "power1.out",
                    y: "1.2rem",
                })
                finishTimeline.set(textNode,{
                    innerHTML:"Bon appetit!"
                })
                finishTimeline.to(textNode,{
                    duration: 0.4,
                    opacity: 1,
                    ease: "power1.out",
                    y: "0",
                })
            }
        }
        /// conclude when routes are finished loading
        function conclude(route: string): void{
            // inefficient code
            let drawSvg, moveSvg;
            for(let i = 0, n = hexagonNode.length; i < n; i++){
                let moveNode = initMoveSvg[i].vars.id
                if(moveNode === `${route}initMoveSvg`) {
                    moveSvg = initMoveSvg[i]
                }
                let drawNode = initDrawSvg[i].vars.id
                if(drawNode === `${route}initDrawSvg`) {
                    drawSvg = initDrawSvg[i]
                }
            }
            let delay = drawSvg.duration() * (1 - drawSvg.progress()) - 0.5
            let t = setTimeout(() => {
                moveSvg.kill()
                moveSvg.targets().forEach(element => {
                    let tl = gsap.timeline({onComplete:drawSvg.kill})
                    tl.to(element,{
                        y: "0",
                        duration:0.5,
                        ease: "power2.inOut"
                    })
                    tl.to(element,{
                        filter: "drop-shadow(0 0 3px white)",
                        duration: 0.5,
                        fill: "rgba(255,255,255,0.8)",
                    })
                });
            },delay * 1000)
            timeout.push(t)
            isLoadedCount++
            if(isLoadedCount == routes.length){
                isLoaded = true
                showPlayButton()
                textTimeline.kill()
                animText()
            }
        }

        /// prefetch routes
        routes.forEach(route => 
            prefetch(route).then(message => {
                if(typeof message.props.page != "undefined"){
                    if(message.props.page.status == 200){
                        conclude(route)
                    }else{
                        console.log("something went wrong")
                    }
                }else{
                    console.log("oh no! an error occured")
                }
            })
        )

        // play
        function showPlayButton(){
            gsap.to(playNode, {
                opacity: 1,
                duration: 0.5,
                delay:0.3,
            })
            gsap.to(playArrowNode,{
                keyframes:{
                    x:[-1, 3],
                },
                duration:0.4,
                repeat:-1,
                yoyo:true,
            })
            gsap.set(playNode, {
                delay:0.2,
                pointerEvents: "all",
            })
        }
    })
    
    // mouse interactions
    function mousePlayButton(action){
        if(action == "enter"){
            gsap.to(playCircleNode,{
                fill: "white",
                filter: "drop-shadow(0 0 3px white)",
                duration:0.3,
            })
            gsap.to(playArrowNode,{
                fill:"black",
            })
        }else if(action == "leave"){
            gsap.to(playArrowNode,{
                fill:"white",
            })
            gsap.to(playCircleNode,{
                fill: "transparent",
                filter: "drop-shadow(0 0 3px transparent)",
                duration:0.3,
            })
        }else if(action == "click"){
            console.log("yahoo!")
            gsap.set(playNode,{
                pointerEvents: "none",
            })
            gsap.set(playCircleNode,{
                strokeDasharray: 25 * Math.PI * 2,
            })
            gsap.to(playNode,{
                scale:1.1,
                duration:1,
                ease:'elastic.out(5, 0.4)'
            })
            gsap.to(textNode,{
                keyframes:[
                    {
                        y:"1rem",
                        duration:0.2,
                    },
                    {
                        display:"none",
                        delay:0.4,
                        duration:0,
                    }
                ]
            })
            gsap.to(textContainerNode,{
                duration: 0.2,
                delay: 0.2,
                height: 0.2,
            })
            gsap.to(playCircleNode,{
                duration:0.5,
                strokeDashoffset: 25 * Math.PI * 2,
                onComplete:function(){
                    gsap.set(playCircleNode,{
                        stroke:'none',
                    })
                    gsap.to(playArrowNode,{
                        scale:0,
                        duration:0.5,
                        onComplete:function(){
                            gsap.set(playCircleNode,{
                                display:"none",
                            })
                        }
                    })
                }
            })
        }
    }
    onDestroy(() =>{
        if(typeof timeout !== "undefined"){
            for(let i = 0, n = timeout.length; i < n; i++){
                clearTimeout(timeout[i])
            }
        }
    })
</script>
<div id="start">
    <div id="text-c">
        <p id="text">a</p>
    </div>
    <div id="hexagon-c">
        <svg class="hexagon" version="1.1" xmlns="http://www.w3.org/2000/svg" fill="rgba(255,255,255,0)" width="26.1" height="30" viewBox="0 0 173.20508075688772 200">
            <path class="hexagon-path" stroke="white" stroke-width="5" d="M86.60254037844386 0L173.20508075688772 50L173.20508075688772 150L86.60254037844386 200L0 150L0 50Z"></path>
        </svg>
        <svg class="hexagon" version="1.1" xmlns="http://www.w3.org/2000/svg" fill="rgba(255,255,255,0)" width="26.1" height="30" viewBox="0 0 173.20508075688772 200">
            <path class="hexagon-path" stroke="white" stroke-width="5" d="M86.60254037844386 0L173.20508075688772 50L173.20508075688772 150L86.60254037844386 200L0 150L0 50Z"></path>
        </svg>
        <svg class="hexagon" version="1.1" xmlns="http://www.w3.org/2000/svg" fill="rgba(255,255,255,0)" width="26.1" height="30" viewBox="0 0 173.20508075688772 200">
            <path class="hexagon-path" stroke="white" stroke-width="5" d="M86.60254037844386 0L173.20508075688772 50L173.20508075688772 150L86.60254037844386 200L0 150L0 50Z"></path>
        </svg>
    </div>
    <div id="play" data-cursor="interact" data-magnet="magnet0.5" on:mouseenter={() => mousePlayButton("enter")} on:mouseleave={() => mousePlayButton("leave")} on:click={() => mousePlayButton("click")}>
        <svg id="play-circle" width="100%" height="100%"><circle id="play-circle-path" cx="50%" cy="50%" r="25" stroke="white" stroke-width="1" fill="none"/></svg>
        <svg id="play-arrow" xmlns="http://www.w3.org/2000/svg" aria-hidden="true" role="img" width="20px" height="20px" fill="white" preserveAspectRatio="xMidYMid meet" viewBox="0 0 24 24"><path d="M13.172 12l-4.95-4.95l1.414-1.414L16 12l-6.364 6.364l-1.414-1.414z" stroke-linecap="round"/></svg>
    </div>
</div>
<style lang="scss">
    #text-c{
        margin-bottom:calc(0.5rem + 7px);
        overflow:hidden;
    }
    #text{
        color:white;
        margin:0 0 1px 0;
        caret-color:transparent;
        transform: translateY(1rem);
    }
    #start{
        position:relative;
        display:flex;
        align-items:center;
        justify-content: center;
        flex-direction:column;
        width:100vw;
        height:100vh;
        overflow:hidden;
    }
    #hexagon-c{
        height:30px;
        width:150px;
        display:flex;
        align-items:center;
        justify-content: center;
        caret-color: transparent;
        svg{
            position:absolute;
            margin:5px;
            opacity: 0;
        }
    }
    #play{
        caret-color: transparent;
        position:relative;
        top:2.5rem;
        width:70px;
        height:70px;
        display:flex;
        align-items:center;
        justify-content: center;
        opacity:0;
        pointer-events:none;
        overflow:hidden;
        svg{
            position:absolute;
            pointer-events:none;
        }
    }
</style>