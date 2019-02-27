---
layout: post
title: "vuejs tinymce hyperlink bug fix"
author: "hoyeonUm"
---


#### 사건의시작
{% highlight html%}
<editor id="editor" v-model="text" ref="tinymce" :readonly="true"></editor>
{% endhighlight %}

다음과 같은 코드가 있었다.
근데 컨텐츠안에 내용이 a 태그의 href 로 링크가 걸려있는 내용이 있었는데 read only 옵션을 줬을때 클릭시 아무런 반응도 하지않는것이다.


{% highlight html%}
<template>
  <div>
      <textarea :id="id">content</textarea> <!--content 에 바인딩처리 필요-->
  </div>
</template>


<script>
import Editor from 'vue-tinymce-editor';
export default {
    extends: Editor,
    methods: {
        initEditor(editor) {
            this.editor = editor;
            editor.on('KeyUp', (e) => {
                this.submitNewContent();
            });
            editor.on('Change', (e) => {
                if(this.editor.getContent() !== this.value){
                    this.submitNewContent();
                }
                this.$emit('editorChange', e);
            });
            editor.on('init', (e) => {
                editor.setContent(this.content);
                this.$emit('input', this.content);
            });
	    //add code
            editor.on('SwitchMode', function() {
                if (editor.readonly) {
                    editor.readonly = 1;
                }
            });
            if(this.readonly){
                this.editor.setMode('readonly');
            } else {
                this.editor.setMode('design');
            }

            this.$emit('editorInit', editor);
        }
    }
}
</script>

{% endhighlight %}
기존 tinymce를 상속받고 SwitchMode 이벤트 함수만 정의해주면 readonly 상태에도 하이퍼링크 클릭이 가능하다.

[참조링크1](https://codepen.io/thorn0/pen/xaYoOb)

[참조링크2](https://stackoverflow.com/questions/52120414/tinymce-how-to-let-user-mark-text-when-readonly-true)
