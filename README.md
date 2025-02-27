# Nurozi
<!DOCTYPE html>
<html lang="fa">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>فال حافظ با موسیقی و گوینده</title>
    <script src="https://aframe.io/releases/1.2.0/aframe.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs"></script>
    <script src="https://cdn.jsdelivr.net/npm/@tensorflow-models/blazeface"></script>
    <script src="https://cdn.jsdelivr.net/npm/face-api.js"></script>
    <style>
        body {
            text-align: center;
            font-family: Arial, sans-serif;
            background-color: black;
            color: white;
        }
        video {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            object-fit: cover;
            opacity: 0.5;
        }
        #fal-text {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            font-size: 24px;
            font-weight: bold;
            background: rgba(0, 0, 0, 0.7);
            padding: 20px;
            border-radius: 10px;
        }
        audio {
            position: absolute;
            bottom: 10px;
            left: 50%;
            transform: translateX(-50%);
            width: 200px;
        }
    </style>
</head>
<body>

    <h2>📷 لبخند بزنید تا فال شما نمایش داده شود!</h2>
    <video id="video" autoplay></video>
    <div id="fal-text">منتظر تشخیص چهره...</div>
    <audio id="background-music" loop>
        <source src="https://www.bensound.com/bensound-music/bensound-relaxing.mp3" type="audio/mpeg">
    </audio>

    <script>
        const falList = {
            happy: [
                "چهره‌ی تو مثل صبحی روشن است! آینده‌ای پر از موفقیت در انتظارت است، پس لبخندت را نگه دار!",
                "لبخندت نشان از بخت خوش دارد، مسیر موفقیت برایت هموار خواهد شد!",
                "امروز هر قدمی که برداری، تو را به آرزوهایت نزدیک‌تر می‌کند!"
            ],
            sad: [
                "غم‌های امروز، خنده‌های فردا را شیرین‌تر می‌کنند. صبور باش، روزگار با تو مهربان خواهد شد.",
                "دنیا همیشه یک‌رنگ نیست، اما هر غروبی طلوعی در پی دارد. امیدت را از دست نده!",
                "هر لحظه سختی که پشت سر می‌گذاری، تو را قوی‌تر می‌کند. ایمان داشته باش که آینده روشنی در انتظار توست."
            ],
            surprised: [
                "زندگی پر از اتفاقات غیرمنتظره است! شاید این لحظه، نقطه عطفی در مسیر تو باشد!",
                "منتظر یک خبر خوب باش! جهان برایت سورپرایزی در نظر دارد!",
                "اتفاقی در راه است که مسیر زندگی تو را تغییر خواهد داد، چشم و دلت را باز نگه دار!"
            ],
            neutral: [
                "آرامش در سادگی است. از لحظاتت لذت ببر و مسیر خودت را با اطمینان ادامه بده.",
                "زندگی تعادل است، نه همیشه خوشی و نه همیشه غم. در این تعادل، تو باید مسیر درست را پیدا کنی.",
                "گاهی هیچ تغییری، بهترین تغییر است. از آرامش این لحظات استفاده کن!"
            ],
            angry: [
                "خشم همچون آتشی است که اول از همه صاحبش را می‌سوزاند. آرام باش و بگذار آرامش راهنمای تو باشد.",
                "گاهی تنها یک نفس عمیق کافی است تا بهترین تصمیم را بگیری.",
                "زندگی کوتاه‌تر از آن است که با عصبانیت بگذرد. رها کن و ادامه بده!"
            ],
            fearful: [
                "ترس‌هایت را بشناس و با آن‌ها روبه‌رو شو، هیچ چیز به اندازه شجاعت تو قدرت ندارد!",
                "اگر امروز نگرانی، فردا دلیل لبخندت را پیدا خواهی کرد!",
                "گاهی ترسیدن یعنی آماده شدن برای یک ماجراجویی جدید! نگران نباش، مسیرت روشن خواهد شد."
            ]
        };

        async function startFaceDetection() {
            const video = document.getElementById('video');
            const falText = document.getElementById('fal-text');
            const music = document.getElementById('background-music');
            const stream = await navigator.mediaDevices.getUserMedia({ video: true });
            video.srcObject = stream;

            await faceapi.nets.tinyFaceDetector.loadFromUri('https://cdn.jsdelivr.net/npm/face-api.js/weights/');
            await faceapi.nets.faceExpressionNet.loadFromUri('https://cdn.jsdelivr.net/npm/face-api.js/weights/');

            // پخش موسیقی پس از دسترسی به دوربین
            music.play();

            setInterval(async () => {
                const detections = await faceapi.detectAllFaces(video, new faceapi.TinyFaceDetectorOptions())
                    .withFaceExpressions();

                if (detections.length > 0) {
                    const expressions = detections[0].expressions;
                    let mood = "neutral";

                    if (expressions.happy > 0.5) mood = "happy";
                    else if (expressions.sad > 0.5) mood = "sad";
                    else if (expressions.surprised > 0.5) mood = "surprised";
                    else if (expressions.angry > 0.5) mood = "angry";
                    else if (expressions.fearful > 0.5) mood = "fearful";

                    let selectedFal = falList[mood][Math.floor(Math.random() * falList[mood].length)];
                    falText.innerText = "✨ فال شما: " + selectedFal;

                    speakText(selectedFal);
                }
            }, 5000);
        }

        function speakText(text) {
            let msg = new SpeechSynthesisUtterance();
            msg.text = text;
            msg.lang = "fa-IR";  
            msg.rate = 0.9;  
            speechSynthesis.speak(msg);
        }

        // شروع شناسایی چهره
        document.getElementById('video').addEventListener('click', startFaceDetection);
    </script>

</body>
</html>
