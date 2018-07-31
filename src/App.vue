  <template>
    <div id="app">

        <h2>TOHO charactor detector(Proto)</h2>
        <div>
            <input type="file" name="file" @change="onFileChange" class="waves-effect waves-light btn">
        </div>
        <div class="buttonwrapper" v-if="image">
          <button @click="removeImage" class="waves-effect waves-light btn">Remove image</button>
          <button @click="submitImage" class="waves-effect waves-light btn">Submit image
            <i class="material-icons right">send</i>
          </button>
        </div>
      <div v-if="image">
        <div class="imagewrapper">
          <img :src="image" />
          <div v-if="predictionData" v-for="prediction in predictionData" :key="prediction.tagID" class="detectionBox" :style="{width:prediction.boundingBox.width*100+'%',height:prediction.boundingBox.height*100+'%',left:prediction.boundingBox.left*100+'%',top:prediction.boundingBox.top*100+'%'}"></div>
        </div>
      </div>
      <div v-if="predictionData" class="tagwrapper">
        <ul class="collection">
          <li class="collection-header">
            <h4>Detected Tags</h4>
          </li>
          <li v-for="prediction in predictionData" :key="prediction.tagID" class="collection-item">
            <b>{{prediction.tagName}}</b>:{{prediction.probability}}
          </li>
        </ul>
      </div>
    </div>
</template>

<script>
import loadImage from 'blueimp-load-image'
import axios from 'axios'

const projectId = "{{ProjectID}}";
const predictionKey = "{{PredictionKey}}";
const postURL = "https://southcentralus.api.cognitive.microsoft.com/customvision/v2.0/Prediction/"+projectId+"/image";
const probabilityLine = 0.15; //確信度閾値


export default {
  name: 'app',
  data: function() {
    return {
      image: '',
      imgName: '',
      imgHeight: '',
      imgWidth:'',
      uploadFile: '',
      predictionData:''
    }
  },
  methods: {
      onFileChange: function(e){
        let files = e.target.files || e.dataTransfer.files;
        if (!files.length) {
            return;
        }
        if (!files[0].type.match('image.*')) {
            return;
        }

        this.createImage(files[0]);
        this.uploadFile = files[0];
        this.predictionData = '';
        this.occupancyRate ='';
        console.log(postURL);
      },
      createImage: function(file) {
        let reader = new FileReader();

        reader.onload = (e) => {
          //ローテーション
          loadImage.parseMetaData(file, (data) => {
            const options = {
              canvas: true
            };
            if (data.exif) {
              options.orientation = data.exif.get('Orientation');
            }
            loadImage(e.target.result, (canvas) => {
              const dataUri = canvas.toDataURL('image/jpeg');
              this.image = dataUri;
              this.imgHeight = canvas.height;
              this.imgWidth = canvas.width;
            }, options);
          });
          //ローテーションここまで
        }
        reader.readAsDataURL(file);
        this.imgName = file.name;
      },
      removeImage: function() {
        this.image = '';
        this.imgName = '';
        this.predictionData = '';
        this.occupancyRate ='';
      },
      submitImage:function() {
        let formData = new FormData();
        formData.append('shelfImage', this.uploadFile);
        let config = {
            method : 'post',
            headers: {
                'content-type': "multipart/form-data",
                'Prediction-Key': predictionKey
                }
        };

        axios.post(postURL, formData, config)
            .then(response => {
                //response 処理
                //確信度フィルタリング
                this.predictionData = response.data.predictions.filter( (items) => Number(items.probability) > probabilityLine);
              })
            .catch(error => {
                  // error 処理
                  this.predictionData = error;
            })
      }
  }
};
</script>

<style>
#app {
    text-align: center;
  }
img {
    width: 100%;
    margin: auto;
    display: block;
    margin-bottom: 10px;
  }
button{
  margin: 10px;
}
.imagewrapper{
  display: inline-block;
  width: 30%;
  position: relative;
  margin: 0px;
  padding: 0px;
  align-self: center;
}
.buttonwrapper{
  padding: 1em;
}
.detectionBox{
  position: absolute;
  border-width: 2px;
  border-style: solid;
  border-color: red;
}
</style>
