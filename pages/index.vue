<template>
  <div class="flex justify-center h-screen">
    <div class="flex flex-col flex-grow max-w-screen-lg">
      <div class="align-middle my-auto">
        <p class="text-center text-5xl px-4" id="titlemsg">Enter some feedback.</p>
        <form class="px-8 pt-6 pb-4 mb-2" ref="form" action="callapi" method="post" @submit.prevent="callapi" id="ideaForm">
          <div class="mb-4">
            <textarea class="text-left shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline" id="description" type="text" placeholder="The movie was alright, but I disliked the ending the most" rows='6'></textarea>
            <div class="text-left shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline hidden" style="overflow-y:scroll; height:150px" id="highlight">
              
            </div>
          </div>
          <div class="flex justify-center">
            <!-- <recaptcha @error="onError" @success="onSuccess" @expired="onExpired" /> -->
            <input class="text-md font-bold pt-2 hover:underline bg-white" type="submit" value="Analyze" id="analbutton"/>
          </div>
        </form>
        <!-- <div> -->
          <!-- <p class="text-center text-green-400 pb-4 hidden" id="success">Idea Submitted!</p> -->
            <!-- <svg class="animate-spin h-5 w-5 mr-3 hidden" id="success" viewBox="0 0 24 24"> -->
            <!-- ... -->
            <!-- </svg> -->
        <!-- </div> -->
      </div>
    </div>
  </div>
</template>

<script>

export default {
    methods: {

      callapi() {
        const inputdata = JSON.stringify({"input": document.querySelector('#description').value});
        fetch("http://35.184.143.105:48932/", {
          method: 'POST',
          body: inputdata
        })
        .then((resp) => resp.json())
        .then(function(response) {
          console.info('fetch()', response);
          var textarea = document.getElementById("description");
          var innerText = textarea.value;
          var cleaninnerText = innerText.toLowerCase();
          cleaninnerText = cleaninnerText.replace(/\n+/g, " ");
          var positions = [];
          for(var idx in response['output_rationales']) {

            const snippet = response['output_rationales'][idx];

            snippet = snippet.replaceAll("#","");
            var pos = cleaninnerText.indexOf(snippet);
            
            if (pos >= 0) {
              positions.push([pos, pos+snippet.length, response['output_logits'][idx]]);
            }
            else
            {
              var minoverlap = Math.floor(0.3*snippet.length);
              var i = 0;
              for(i = 0; i < snippet.length-minoverlap; i++) {
                if(cleaninnerText.indexOf(snippet.substring(i, i+minoverlap)) >= 0)
                  break;
              }
              if(i < snippet.length-minoverlap) {
                pos = cleaninnerText.indexOf(snippet.substring(i, i+minoverlap));
                positions.push([pos-i, pos-i+snippet.length, response['output_logits'][idx]]);
              }
              // var minoverlap = Math.floor(0.3*snippet.length);
              // var maxconsec = 0;
              // var ct = 0;
              // var pos0 = 0;
              // for(var i = 0; i < snippet.length-minoverlap; i++) {
              //   var newpos = cleaninnerText.indexOf(snippet.substring(i, i+minoverlap));
              //   if(newpos == pos+1) {
              //     ct++; 
              //   }
              //   else {
              //     if(ct > maxconsec) {
              //       maxconsec = ct;
              //       pos0 = i-ct;
              //     }
              //     ct = 0;
              //   }
              //   pos = newpos;
              // }
              // if(maxconsec > 0) {
              //   positions.push(pos0);
              //   positions.push(pos0 + maxconsec);
              // }
            }
            // console.log(response['output_rationales'][item]);
          }
          
          positions.sort(function(x,y)
          {
            if(x[0] < y[0])
              return -1;
            else if(x[0] > y[0])
              return 1;
            else
              return x[1] - y[1];
          });

          var ct = 0;
          var newtext = "";
          // console.log(cleaninnerText);
          for(var i = 0; i < innerText.length; i++) {
            if(ct < positions.length && i == positions[ct][0]) {
              if(positions[ct][2]==0)
                newtext += "<span style=\"background-color: #FF9494\">";
              if(positions[ct][2]==1)
                newtext += "<span style=\"background-color: #FFC094\">";
              if(positions[ct][2]==2)
                newtext += "<span style=\"background-color: #F7FF94\">";
              if(positions[ct][2]==3)
                newtext += "<span style=\"background-color: #C5FBA9\">";
              if(positions[ct][2]==4)
                newtext += "<span style=\"background-color: #A9FBB5\">";
            }
            else if(ct < positions.length && i == positions[ct][1]) {
                newtext += "</span>";
                ct += 1;
            }
            newtext = newtext.concat(innerText.charAt(i));
          }
          //textarea.style.display = 'none';
          //console.log(newtext);
          document.getElementById("highlight").innerHTML = newtext;
          document.getElementById("highlight").classList.remove("hidden");
          document.getElementById("description").classList.add("hidden");
          document.getElementById("analbutton").classList.add("hidden");
          document.getElementById("titlemsg").innerText = "Here's your analysis!"
        });
      }
        // call stuff
        // document.getElementById("success").classList.remove("hidden");
    }
}


</script>

<style>

</style>
