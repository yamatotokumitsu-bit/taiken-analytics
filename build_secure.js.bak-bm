const fs=require('fs'),crypto=require('crypto');
const pass=process.argv[2], src=process.argv[3], out=process.argv[4];
let html=fs.readFileSync(src,'utf8');
const m=html.match(/([\s\S]*?)<script>([\s\S]*?)<\/script>([\s\S]*)$/);
if(!m){console.error("no script");process.exit(1);}
const head=m[1], body=m[2], tail=m[3];
// 行ベースで const D=<json>; を分離
const lines=body.split('\n');
let di=lines.findIndex(l=>l.trimStart().startsWith('const D='));
if(di<0){console.error("no const D line");process.exit(1);}
let dline=lines[di].trim();
if(!dline.endsWith(';')){console.error("const D line not single-line");process.exit(1);}
const json=dline.slice('const D='.length,-1);
JSON.parse(json); // 検証（壊れてたら例外で停止）
const rest=lines.slice(0,di).concat(lines.slice(di+1)).join('\n'); // const D行を除いた残り全部
// 暗号化 AES-256-GCM + PBKDF2
const salt=crypto.randomBytes(16), iv=crypto.randomBytes(12);
const key=crypto.pbkdf2Sync(pass,salt,100000,32,'sha256');
const c=crypto.createCipheriv('aes-256-gcm',key,iv);
const enc=Buffer.concat([c.update(json,'utf8'),c.final()]);
const tag=c.getAuthTag();
const CIPHER={s:salt.toString('base64'),i:iv.toString('base64'),d:Buffer.concat([enc,tag]).toString('base64')};
const gate=`
<div id="__gate" style="position:fixed;inset:0;z-index:9999;background:#f9f8f4;color:#191713;display:flex;align-items:center;justify-content:center;font-family:system-ui,-apple-system,'Hiragino Sans',sans-serif">
 <form id="__gform" style="background:#fff;border:1px solid rgba(0,0,0,.12);border-radius:14px;padding:30px 34px;max-width:380px;width:90%;box-shadow:0 10px 40px rgba(0,0,0,.15)">
  <div style="font-size:12px;letter-spacing:.16em;color:#8a6d1f;font-weight:700">TOKYO YAMAGAWA DMC</div>
  <h1 style="font-size:20px;margin:6px 0 4px;font-family:'Hiragino Mincho ProN',serif">体験販売 分析スタジオ</h1>
  <p style="font-size:12.5px;color:#666;margin:0 0 16px;line-height:1.7">社内限定の資料です。合言葉を入力してください。</p>
  <input id="__pw" type="password" placeholder="合言葉" autocomplete="off" autofocus style="width:100%;padding:11px 13px;font-size:15px;border:1px solid #ccc;border-radius:8px;box-sizing:border-box">
  <button type="submit" style="width:100%;margin-top:12px;padding:11px;font-size:14px;font-weight:700;border:none;border-radius:8px;background:#233d33;color:#fff;cursor:pointer">ひらく</button>
  <p id="__err" style="display:none;font-size:12px;color:#b91c1c;margin:10px 0 0">合言葉がちがうようです。もう一度お試しください。</p>
 </form>
</div>`;
const decryptJS=`
const CIPHER=${JSON.stringify(CIPHER)};
function __b64(s){const b=atob(s),u=new Uint8Array(b.length);for(let i=0;i<b.length;i++)u[i]=b.charCodeAt(i);return u;}
async function __unlock(pw){
 const enc=new TextEncoder();
 const km=await crypto.subtle.importKey('raw',enc.encode(pw),'PBKDF2',false,['deriveKey']);
 const key=await crypto.subtle.deriveKey({name:'PBKDF2',salt:__b64(CIPHER.s),iterations:100000,hash:'SHA-256'},km,{name:'AES-GCM',length:256},false,['decrypt']);
 const plain=await crypto.subtle.decrypt({name:'AES-GCM',iv:__b64(CIPHER.i)},key,__b64(CIPHER.d));
 return JSON.parse(new TextDecoder().decode(plain));
}
document.getElementById('__gform').addEventListener('submit',async e=>{
 e.preventDefault();
 const btn=e.target.querySelector('button');btn.textContent='ひらいています…';
 try{ D=await __unlock(document.getElementById('__pw').value.trim());
  const g=document.getElementById('__gate'); if(g) g.remove(); __boot();
 }catch(_){ document.getElementById('__err').style.display='block';
  btn.textContent='ひらく';
  document.getElementById('__pw').value='';document.getElementById('__pw').focus(); }
});`;
const newBody=`let D=null;\n${decryptJS}\nfunction __boot(){\n${rest}\n}`;
const noindex='<meta name="robots" content="noindex,nofollow">\n';
const finalHtml=noindex+head+gate+'\n<script>\n'+newBody+'\n</script>'+tail;
fs.writeFileSync(out,finalHtml);
console.log("OK built:",out,"|",finalHtml.length,"bytes | JSON",json.length,"chars encrypted");
