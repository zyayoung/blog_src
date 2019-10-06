---
title: 拯救被压爆的表情包Demo
categories: CV
date: 2019-10-07 00:18:00
---

Demo

<!-- more -->

{% raw %}<input type="file" id="fileInput" name="file" accept="image/*">{% endraw %}

Input:

{% raw %}<img id="imageSrc" alt="No Image" hidden>{% endraw %}

Output:

{% raw %}<canvas style="width:100%" id="canvasOutput"></canvas>{% endraw %}

{% raw %}
    <!-- jQuery (necessary for Bootstrap's JavaScript plugins) -->
    <script src="https://cdn.bootcss.com/jquery/1.12.4/jquery.min.js"></script>
    <!-- Bootstrap Select -->
    <script src="https://cdn.bootcss.com/popper.js/1.14.6/umd/popper.min.js"></script>
    <!-- Include all compiled plugins (below), or include individual files as needed -->
    <script src="https://cdn.bootcss.com/twitter-bootstrap/4.2.1/js/bootstrap.min.js"></script>
    <script src="https://cdn.bootcss.com/bootstrap-select/1.13.5/js/bootstrap-select.min.js"></script>
    <!-- <script src="opencv.js" type="text/javascript"></script> -->
    <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@1.0.0/dist/tf.min.js"></script>
    <script type="text/javascript">
        let imgElement = document.getElementById('imageSrc');
        let inputElement = document.getElementById('fileInput');
        let outputCanvas = document.getElementById('canvasOutput');
        inputElement.addEventListener('change', (e) => {
            imgElement.src = URL.createObjectURL(e.target.files[0]);
        }, false);
        tf.setBackend('webgl');
        imgElement.onload = async function () {
            const model = await tf.loadLayersModel('https://raw.githubusercontent.com/zyayoung/FixJPG/master/tfjs_model/model.json');
            const warmupResult = model.predict(tf.zeros([1, 32, 32, 1]));
            warmupResult.dataSync();
            warmupResult.dispose();
            let im = tf.browser.fromPixels(imgElement);
            let x = im.expandDims(0).transpose([3,1,2,0]).div(255);
            let result = model.predict(x).clipByValue(0, 1).mul(255).transpose([3,1,2,0]).squeeze().asType('int32');
            tf.browser.toPixels(result, outputCanvas);
        };
    </script>
{% endraw %}

