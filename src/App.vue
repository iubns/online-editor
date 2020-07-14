<template>
  <v-app >
    <div ref="app" style="height:calc(100% - 112px);">
      <v-toolbar color="#36435B">
        <v-btn text small color="white" @click="$refs.file.click()">
          <div class="body-2">
          Open File 
          </div>
          <input type="file" ref="file" style="display: none" @change="handleChange">
        </v-btn>
        <v-select
          v-model="language"
          :items="languages"
          item-text="display"
          item-value="value"
          style="margin-top:30px;"
          solo
        >
        </v-select>
        <v-spacer></v-spacer>
        <v-btn text small color="white" @click="() => IsFullScreen = !IsFullScreen">
          <div class="body-2">
            {{ IsFullScreen ? 'normal' : 'pull' }} screen
          </div>
        </v-btn>
      </v-toolbar>

      <Editor
        class="my-editor"
        v-model="sourcecode"
        :line-numbers="true"
        :language="language.trim()"
      />

      <transition name="bottom">
        <div class="bottom" ref="bottom" v-if="!IsConsoleClose" >
          <v-toolbar color="#36435B">
            <v-btn text small color="white" v-bind:class="{clickedButton: (bottomStatus === 'Input')}" @click="() => bottomStatus = 'Input'" >
              <div class="body-2">
                Input
              </div>
            </v-btn>
            <v-btn text small color="white" v-bind:class="{clickedButton: (bottomStatus === 'Output')}" @click="() => bottomStatus = 'Output'">
              <div class="body-2">
                Output
              </div>
            </v-btn>
            <v-btn text small color="white" v-bind:class="{clickedButton: (bottomStatus === 'Stderr')}" @click="() => bottomStatus = 'Stderr'">
              <div class="body-2">
                Stderr
              </div>
            </v-btn>
            <v-btn text small color="white" v-bind:class="{clickedButton: (bottomStatus === 'Compilation')}" @click="() => bottomStatus = 'Compilation'">
              <div class="body-2">
                Compilation
              </div>
            </v-btn>
            <v-btn text small color="white" v-bind:class="{clickedButton: (bottomStatus === 'Execution')}" @click="() => bottomStatus = 'Execution'">
              <div class="body-2">
                Execution
              </div>
            </v-btn>

            <v-spacer></v-spacer>
            <v-btn text small color="white" @click="(() => {this.IsConsoleClose = true})">
              <div class="body-2">
                Close
              </div>
            </v-btn>
          </v-toolbar>
          <Editor
            class="bottomConsole"
            v-if="bottomStatus === 'Input' || bottomStatus === 'Output' || bottomStatus === 'Stderr' "
            v-model="bottomData[bottomStatus]"
            :line-numbers="true"
            language="text"
          />

          <div v-else-if="bottomStatus === 'Compilation'">
            {{ CompilationMessage }}
          </div>
          <div v-else-if="bottomStatus === 'Execution'">
            <v-row dense>
              <v-card>
                <div class="cardTop">
                  CPU Time
                </div>
                <div class="cardBottom">
                  {{ cpuTime }}
                </div>
              </v-card>
              <v-card>
                <div class="cardTop">
                  Memory Usage
                </div>
                <div class="cardBottom">
                  {{ MemoryUsage }}
                </div>
              </v-card>
              <v-card>
                <div class="cardTop">
                  Exit Code
                </div>
                <div class="cardBottom">
                  {{ ExitCode }}
                </div>
              </v-card>
            </v-row>
          </div>

        </div>
      
      </transition>
      <v-toolbar color="#36435B">
        <v-btn small color="primary" style="margin: 10px; float:right;" @click="() => {IsConsoleClose = !IsConsoleClose}">{{IsConsoleClose ? 'Opne' : 'Close'}} Console</v-btn>
          <v-spacer></v-spacer>
        <v-btn small color="primary" style="margin: 10px; float:right;">Compile</v-btn>
        <v-btn small color="primary" style="margin: 10px; float:right;">submit</v-btn>
      </v-toolbar>
    </div>
  </v-app>
</template>

<script>
/* global Prism */
import "prismjs";
import "./assets/editorstyle.css";
import 'prismjs/components/prism-c'
import 'prismjs/components/prism-cpp'
import 'prismjs/components/prism-java'
import 'prismjs/components/prism-python'
// import "prismjs/themes/prism-tomorrow.css";
import Editor from "./components/Editor.vue";

export default {
  name: "app",
  components: {
    Editor
  },
  data: () => ({
    lineNumbers: true,
    readonly: false,
    sourcecode: '#include <iostream>\n\n main(){\n\tstd::cout << "hello";\n\treturn 0;\n}',
    bottomData: [],
    bottomStatus: 'Input',
    language: 'c',
    languages: [
      {display: 'C', value :'c'},
      {display: 'CPP', value :'cpp'},
      {display: 'python', value :'python'},
      {display: 'JAVA', value :'java'}
    ],
    IsConsoleClose: false,
    IsFullScreen: false,
    CompilationMessage: '',
    cpuTime:'',
    MemoryUsage:'',
    ExitCode:''
  }),
  mounted() {
  },
  methods:{
    async handleChange(e) {
      e.preventDefault()
      const files = e.target.files || e.dataTransfer.files
        if(files[0].size > 1024 * 60){
          alert('파일의 크기가 너무 큽니다.')
          return;
        }
        
        const reader = new FileReader()
        await new Promise((resolve, reject) => {
          reader.onloadend = function(e) {
            const content = e.target.result;
            const result = content.split(/\r\n|\n/);
            resolve(result);
          };
          reader.onerror = function(e) {
            reject(e);
          };
          reader.readAsText(files[0])
        })
        this.sourcecode = reader.result
        this.$refs.file.value = ''
    },
    async compile(){
      await axios.post('',{
        language: this.language,
        code: this.sourcecode,
        stdin: this.bottomData['Input']
      })
    }
  },
  watch:{
    language(newValue){
      switch(newValue){
          case 'c':
          this.sourcecode = '#include <stdio.h>\nint main() {\n\n\tprintf("Hello, World!");\n\treturn 0;\n}'
        break
          case 'cpp':
          this.sourcecode = '#include <iostream>\n\nint main(){\n\tstd::cout << "Hello, World!";\n\treturn 0;\n}'
        break
          case 'python':
          this.sourcecode = 'print("Hello, World!")'
        break
          case 'java':
          this.sourcecode = 'public class Main {\n\tpublic static void main(String[] args) {\n\t\tSystem.out.println("Hello, World!");\n\t}\n}'
        break
      }
    },
    IsFullScreen(newValue){
      if(newValue){
        this.$refs.app.webkitRequestFullscreen()
      }else{
        document.webkitExitFullscreen()
      }
    }
  }
};
</script>


<style>
.my-editor {
  max-height: 950px;
}
.bottomConsole{
  max-height: 300px;
}
.bottom {
  position: absolute;
  bottom: 110px;
  max-height: 300px;
  height: 100%;
  width: 100%;
}
.bottom-enter-active {
  animation-name: bottom-in;
  animation-duration: 0.7s;
  animation-fill-mode: forwards;
  animation-direction: reverse;
}
.bottom-leave-active {
  animation-name: bottom-in;
  animation-duration: 0.7s;
  animation-fill-mode: forwards;
  animation-timing-function: linear;
}
.clickedButton{
  background-color: #222C40 !important;
  color: white !important;
}
.v-card{
  width: 250px;
  height: 80px;
  margin: 20px auto;
}
.cardTop{
  background-color: black;
  color: white;
  height: 40px;
  width: 100%;
  padding-top: 8px;
  text-align: center;
}
.cardBottom{
  padding-top: 8px;
  text-align: center;
}

@keyframes bottom-in {
  0% {
    transform: translate(0, 0px);
    height: 100%;
  }
  20%{
    transform: translate(0, 20%);
    height: 10%;
  }
  100%{
    transform: translate(0, 100%);
    height: 0%;
  }
}
</style>
