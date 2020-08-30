//Anime4K v3.1 GLSL

// This is free and unencumbered software released into the public domain.

// Anyone is free to copy, modify, publish, use, compile, sell, or
// distribute this software, either in source code form or as a compiled
// binary, for any purpose, commercial or non-commercial, and by any
// means.

// In jurisdictions that recognize copyright laws, the author or authors
// of this software dedicate any and all copyright interest in the
// software to the public domain. We make this dedication for the benefit
// of the public at large and to the detriment of our heirs and
// successors. We intend this dedication to be an overt act of
// relinquishment in perpetuity of all present and future rights to this
// software under copyright law.

// THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
// EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
// MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
// IN NO EVENT SHALL THE AUTHORS BE LIABLE FOR ANY CLAIM, DAMAGES OR
// OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE,
// ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR
// OTHER DEALINGS IN THE SOFTWARE.

// For more information, please refer to <https://unlicense.org>

//!DESC Anime4K-v3.1-DarkLines-Kernel(X)
//!HOOK NATIVE
//!BIND HOOKED

vec3 rgb2hsv(vec3 c)
{
    vec4 K = vec4(0.0, -1.0 / 3.0, 2.0 / 3.0, -1.0);
    vec4 p = mix(vec4(c.bg, K.wz), vec4(c.gb, K.xy), step(c.b, c.g));
    vec4 q = mix(vec4(p.xyw, c.r), vec4(c.r, p.yzx), step(p.x, c.r));

    float d = q.x - min(q.w, q.y);
    float e = 1.0e-10;
    return vec3(abs(q.z + (q.w - q.y) / (6.0 * d + e)), d / (q.x + e), q.x);
}

vec3 hsv2rgb(vec3 c)
{
    vec4 K = vec4(1.0, 2.0 / 3.0, 1.0 / 3.0, 3.0);
    vec3 p = abs(fract(c.xxx + K.xyz) * 6.0 - K.www);
    return c.z * mix(K.xxx, clamp(p - K.xxx, 0.0, 1.0), c.y);
}

vec3 sat(vec3 img)
{
    img = img / 255.0;
    float Increment  = 0.03;
    float img_min = min(img.r, min(img.g, img.b));
    float img_max = max(img.r, max(img.g, img.b));

    float Delta = (img_max - img_min);
    float value = (img_max + img_min);

    float L = value/2.0;

    float mask_1 = 0.0;
    if(L < 0.5) {
        mask_1 = 1.0;
    } else {
        mask_1 = 0.0;
    }

    float s1 = Delta / (value + 0.001);
    float s2 = Delta / (2.0 - value + 0.001);
    float s = s1*mask_1 + s2*(1.0 - mask_1);


    if(Increment >= 0.0) {
        float temp = Increment + s;
        float mask_2 = 0.0;
        if(temp > 1.0){
            mask_2 = 1.0;
        } else {
            mask_2 = 0.0;
        }
        float alpha_1 = s;
        float alpha_2 = s*0.0 + 1.0 - Increment;
        float alpha = alpha_1*mask_2 + alpha_2*(1.0 - mask_2);
        alpha = 1.0/(alpha + 0.001) - 1.0;
        img.r = img.r + (img.r - L) * alpha;
        img.g = img.g + (img.g - L) * alpha;
        img.b = img.b + (img.b - L) * alpha;
    } else {
        float alpha = Increment;
        img.r = img.r + (img.r - L) * alpha;
        img.g = img.g + (img.g - L) * alpha;
        img.b = img.b + (img.b - L) * alpha;        
    }


    vec3 img_out = img;
    float mask_r_1 = 0.0;
    float mask_r_2 = 0.0;
    float mask_g_1 = 0.0;
    float mask_g_2 = 0.0;
    float mask_b_1 = 0.0;
    float mask_b_2 = 0.0;

    if(img_out.r < 0.0){
        mask_r_1 = 1.0;
    } else {
        mask_r_1 = 0.0;
    }
    if(img_out.r > 1.0){
        mask_r_2 = 1.0;
    } else {
        mask_r_2 = 0.0;
    }

    if(img_out.g < 0.0){
        mask_g_1 = 1.0;
    } else {
        mask_g_1 = 0.0;
    }
    if(img_out.g > 1.0){
        mask_g_2 = 1.0;
    } else {
        mask_g_2 = 0.0;
    }
     

    if(img_out.b < 0.0){
        mask_b_1 = 1.0;
    } else {
        mask_b_1 = 0.0;
    }
    if(img_out.b > 1.0){
        mask_b_2 = 1.0;
    } else {
        mask_b_2 = 0.0;
    }


    //img_out = img_out * (1.0-mask_r_1, 1.0-mask_g_1, 1.0-mask_b_1);
    img_out.r = img_out.r*(1.0-mask_r_1);
    img_out.g = img_out.g*(1.0-mask_g_1);
    img_out.b = img_out.b*(1.0-mask_b_1);
    //img_out = img_out * (1.0-mask_r_2, 1.0-mask_g_2, 1.0-mask_b_2) + (mask_r_2, mask_g_2, mask_b_2);
    img_out.r = img_out.r*(1.0-mask_r_2);
    img_out.g = img_out.g*(1.0-mask_g_2);
    img_out.b = img_out.b*(1.0-mask_b_2);
    img_out += (mask_r_2, mask_g_2, mask_b_2);
    img_out = img_out * 255.0;

    return img_out;
}

#define L_tex HOOKED_tex

#define SIGMA 1

float gaussian(float x, float s, float m) {
	return (1 / (s * sqrt(2 * 3.14159))) * exp(-0.5 * pow(abs(x - m) / s, 2.0));
}

float lumGaussian(vec2 pos, vec2 d) {
	float s = SIGMA * HOOKED_size.y / 1080;
	float kernel_size = s * 2 + 1;
	
	float g = (L_tex(pos).x) * gaussian(0, s, 0);
	float gn = gaussian(0, s, 0);
	
	g += (L_tex(pos - d).x + L_tex(pos + d).x) * gaussian(1, s, 0);
	gn += gaussian(1, s, 0) * 2;
	
	for (int i=2; i<kernel_size; i++) {
		g += (L_tex(pos - (d * i)).x + L_tex(pos + (d * i)).x) * gaussian(i, s, 0);
		gn += gaussian(i, s, 0) * 2;
	}
	
	return g / gn;
}

float getLum(vec4 rgb) {
	return (rgb.r + rgb.r + rgb.g + rgb.g + rgb.g + rgb.b) / 6.0;
}

vec4 hook() { //Save lum on OUTPUT
	vec4 rgb = HOOKED_tex(HOOKED_pos);
    rgb.rgb = sat(rgb.rgb);
    //rgb.rgb = rgb.rgb / 255.0;
    //rgb.rgb = rgb.rgb * 255.0;
    return rgb; //vec4(255.0); //vec4(lum);
}
