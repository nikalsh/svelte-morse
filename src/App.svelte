<script>

    String.prototype.insert = function (index, string) {
        if (index > 0) {
            return this.substring(0, index) + string + this.substring(index, this.length);
        }
        return string + this;
    };

    String.prototype.replaceAt = function (index, string) {
        if (index > this.length - 1) return this.insert(index, string);
        return this.substr(0, index) + string + this.substr(index + 1);
    }

    function TreeNode(val, left, right) {
        this.val = val;
        this.left = left;
        this.right = right;
    }

    let morse = new TreeNode(null,
            new TreeNode("T",
                    new TreeNode("M",
                            new TreeNode("O",
                                    new TreeNode(null,
                                            new TreeNode("0", null, null),
                                            new TreeNode("9", null, null)),
                                    new TreeNode("8", null, null)),
                            new TreeNode("G",
                                    new TreeNode("Q", null, null),
                                    new TreeNode("Z", null,
                                            new TreeNode("7", null, null)))),
                    new TreeNode("N",
                            new TreeNode("K",
                                    new TreeNode("Y", null, null),
                                    new TreeNode("C", null, null)),
                            new TreeNode("D",
                                    new TreeNode("X", null, null),
                                    new TreeNode("B", null,
                                            new TreeNode("6", null, null))))),
            new TreeNode("E",
                    new TreeNode("A",
                            new TreeNode("W",
                                    new TreeNode("J",
                                            new TreeNode("1", null, null), null),
                                    new TreeNode("P", null, null)),
                            new TreeNode("R", null,
                                    new TreeNode("L", null, null))),
                    new TreeNode("I",
                            new TreeNode("U",
                                    new TreeNode("-",
                                            new TreeNode("2", null, null), null),
                                    new TreeNode("F", null, null)),
                            new TreeNode("S",
                                    new TreeNode("V",
                                            new TreeNode("3", null, null)),
                                    new TreeNode("H",
                                            new TreeNode("4", null, null),
                                            new TreeNode("5", null, null)))))
    )

    function traverse(node) {
        if (!!node && !!node.left) {
            traverse(node.left);
        }
        if (!!node && !!node.right) {
            traverse(node.right);
        }
        if (node.val !== null) {
            console.log(node.val);
        }
    }

    function handleClick() {
        traverse(morse);
    }

    function reset() {
        morseTxt = "";
        alphaTxt = "";
        morseChar = "";
        lastPress = null;
        c = 0;
        document.getElementById("resetButton").blur();
    }

    const welcome = "Tap space to start typing"
    let start = null;
    let end = null;
    let delta = null;
    let lastPress = null;
    let c = 0;
    let t = null;
    let timer = null;
    let oscillatorStarted = false;
    let morseTxt = "";
    let morseChar = "";
    let alphaTxt = " ";

    let AudioContext = window.AudioContext || window.webkitAudioContext;
    let ctx = new AudioContext();
    let oscillator = ctx.createOscillator();
    oscillator.type = "sine";
    oscillator.frequency.value = 600;
    let gainNode = ctx.createGain();
    oscillator.connect(gainNode);
    gainNode.connect(ctx.destination);

    document.onkeydown = function (e) {
        if (e.key === " ") {
            if (start === null) {
                start = performance.now();
                t = ctx.currentTime;
            }

            gainNode.gain.setValueAtTime(0.2, t);
            if (!oscillatorStarted) {
                oscillator.start();
                oscillatorStarted = true;
            }
        }
    };

    document.onkeyup = function (e) {
        if (e.key === " ") {
            clearInterval(timer);

            end = performance.now();
            delta = end - start;

            if (delta <= 125) {
                morseChar += ".";
                morseTxt += ".";
            } else {
                morseChar += "-";
                morseTxt += "-";
            }

            alphaTxt = alphaTxt.replaceAt(c, translateMorse(morseChar));
            lastPress = performance.now();

            timer = setInterval(function () {
                if (performance.now() - lastPress >= 420) {
                    alphaTxt = alphaTxt.replaceAt(c, translateMorse(morseChar));
                    c++;
                    morseChar = "";
                    morseTxt += " ";
                    clearInterval(timer);
                }
            }, 1);

            t += delta / 1000;
            gainNode.gain.setValueAtTime(0, t);
            start = null;
            t = null;
        }
    }

    function translateMorse(chars) {
        let arr = chars.split("");
        let node = morse;

        for (let i = 0; i < arr.length; i++) {
            if (arr[i] === "." && !!node.right) {
                node = node.right;
            }

            if (arr[i] === "-" && !!node.left) {
                node = node.left;
            }
        }

        if (!!node.val) {
            return node.val;
        } else {
            return " ";
        }
    }

</script>

<main>
    {welcome}
    <div class="container">
        <div class="text">
            {morseTxt}
        </div>
        <div class="text">
            {alphaTxt}
        </div>
    </div>
    <div class="container">
        <button id="resetButton" on:click={reset}>Reset</button>
        <div class="container imgcontainer">
            <img src="http://www.learnmorsecode.com/pix/learn.gif" alt="morse">
        </div>
    </div>
</main>

<style>
    .imgcontainer {
        margin-top: 100px;
    }

    .container {
        padding: 15px;
        display: block;
    }

    .text {
        text-align: center;
        max-width: 100%;
        min-height: 24px;
        display: block;
    }

    main {
        text-align: center;
        padding: 1em;
        max-width: 240px;
        margin: 0 auto;
        word-break: break-all;
        background-color: white;
    }

    h1 {
        color: #ff3e00;
        text-transform: uppercase;
        font-size: 4em;
        font-weight: 100;
    }

    @media (min-width: 640px) {
        main {
            max-width: none;
        }
    }
</style>