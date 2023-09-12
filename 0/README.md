# 【シェーダー解説 #0】ルーン文字シェーダー

## リンク

[twigl](https://twigl.app/)
[youtube](https://youtu.be/XsAXXQ0V9kk?si=8c7K6jV-ktROr50L)

## 補足

線を描画する箇所で`float(sd<0.05)`としていましたが、これを`smoothstep(0.05,0.04,sd)`に変えるとより線が滑らかになります

## コード

https://twigl.app?ol=true&ss=-Ne7rRxlGMi0dZkhgb4N

```glsl
precision highp float;
uniform vec2 resolution;
uniform vec2 mouse;
uniform float time;
uniform sampler2D backbuffer;
out vec4 outColor;

// Hash without Sine by David Hoskins.
// https://www.shadertoy.com/view/4djSRW
///  2 out, 2 in...
vec2 hash22(vec2 p)
{
    vec3 p3 = fract(vec3(p.xyx) * vec3(.1031, .1030, .0973));
    p3 += dot(p3, p3.yzx + 33.33);
    return fract((p3.xx + p3.yz) * p3.zy);
}

// https://iquilezles.org/articles/distfunctions2d/
float sdSeg(vec2 p, vec2 a, vec2 b)
{
    vec2 pa = p - a, ba = b - a;
    float h = clamp(dot(pa, ba) / dot(ba, ba), 0.0, 1.0);
    return length(pa - ba * h);
}

float rune(vec2 uv,float seed)
{
  const float div=3.;
  const int n=5;
  float c=0.0;
  for(int i;i<n;i++)
  {
    vec2 p0 = hash22(vec2(i,seed));
    vec2 p1 = hash22(vec2(i+1,seed));
    p0=(floor(p0*div)+.5)/div;
    p1=(floor(p1*div)+.5)/div;
    float sd=sdSeg(uv,p0,p1);
    float isdraw=float(hash22(vec2(seed,i)).x<0.5);
    
    c+=float(sd<0.05)*isdraw;
  }
  return clamp(c,0.0,1.0);
}


void main(){
  vec2 fc=gl_FragCoord.xy,res=resolution.xy;
  vec2 uv=fc/res.y*10.0;
  uv.y+=time;
  vec2 uvf=fract(uv);
  vec2 uvi=floor(uv);
  
  float c=rune(uvf,hash22(uvi).x);
  vec3 col=vec3(c);
  outColor=vec4(col,1);
}
```