# DEVLOG — InfluencerOS + ugc-factory
> Gerado automaticamente. Última atualização: 2026-05-15
> **Para outro Claude:** Este documento contém o estado completo e exato do projeto. Siga as instruções para reconstruir 100% do que existe aqui.

---

## 🧠 BRIEFING PARA OUTRO CLAUDE — leia isso primeiro

### O que é este projeto
**InfluencerOS** = plataforma para criar influencers de IA para TikTok/Shopee/Instagram.
- `f:\influenceros` — Next.js frontend (porta 3000)
- `f:\ugc-factory` — Python FastAPI backend com agentes de IA (porta 8000)

### O que fizemos nesta sessão (mudanças reais aplicadas)

| # | Arquivo | O que mudou |
|---|---------|-------------|
| 1 | `f:\ugc-factory\src\services\tank.py` | Adicionou função `enhance_image()` — Clarity Upscaler 2x via fal-ai/clarity-upscaler |
| 2 | `f:\ugc-factory\src\api\routes.py` | Adicionou `EnhanceImageRequest` + endpoint `POST /enhance-image` |
| 3 | `f:\influenceros\app\api\influencers\enhance-image\route.ts` | **CRIADO** — proxy Next.js → ugc-factory /enhance-image |
| 4 | `f:\influenceros\app\(app)\influencers\new\page.tsx` | Adicionou estados `genTiming`, `enhancing`; função `handleEnhance`; timing badge em cada imagem; botão "B" (Clarity) no hover; `genStartTime` no handleGenerate |
| 5 | `f:\influenceros\app\(app)\influencers\new\page.tsx` | LoRA training steps: `1000` → `2500` |

### O que NÃO foi aplicado ainda (pendente)

```
FIX CRÍTICO — cabelo/pele mudam na geração:
  genMode="kontext" chama generate_image_pulid() (só preserva geometria facial)
  Devia chamar generate_image_kontext() (edita in-place, preserva loiro/pele)

  3 arquivos para editar:
  1. f:\influenceros\app\api\influencers\generate-image\route.ts
  2. f:\ugc-factory\src\api\routes.py
  3. f:\influenceros\app\(app)\influencers\new\page.tsx
  (ver seção "Fix Pendente" abaixo)
```

### Agentes do ugc-factory (não alterar)
- `agent_brown.py` → analisa foto, extrai DNA facial em JSON estruturado
- `trinity.py` → gera prompt de 7 blocos para geração de imagem
- `oracle.py` → gera prompt de 6 blocos para vídeo Veo3
- `cypher.py` → virtual try-on via fashn/tryon
- `refiner.py` → compara referência vs gerado, sugere correções de prompt

### APIs externas usadas
- **FAL.ai** — flux-pulid, flux-kontext, instantid, clarity-upscaler, veo3, flux-lora, flux-lora-portrait-trainer
- **Anthropic Claude** — análise de imagem (Brown), geração de prompts (Trinity, Oracle, Refiner)
- **Supabase** — auth, storage, banco de dados

---

## ⚠️ PROBLEMA ATUAL: Conexão com API

**Sintoma:** Frontend não consegue se comunicar com ugc-factory ou APIs externas.

**Diagnóstico — cheque nesta ordem:**

```powershell
# 1. ugc-factory está rodando?
Invoke-WebRequest http://localhost:8000/docs -UseBasicParsing | Select-Object StatusCode
# Deve retornar 200. Se não, iniciar:
cd f:\ugc-factory
.\.venv\Scripts\uvicorn.exe api:app --port 8000 --reload

# 2. Next.js está rodando?
Invoke-WebRequest http://localhost:3000 -UseBasicParsing | Select-Object StatusCode

# 3. FAL API key válida?
# Verificar f:\ugc-factory\.env — FAL_API_KEY deve começar com "fal-"

# 4. CORS: ugc-factory aceita origem 3000?
# Verificar f:\ugc-factory\api.py — allow_origins deve incluir "http://localhost:3000"
```

**O que fazer sem conexão:**
- Editar código (todos os arquivos abaixo estão documentados)
- Fazer commits git
- Ler e ajustar prompts dos agentes
- Configurar variáveis de ambiente

---

## Arquitetura do Sistema

```
influenceros (Next.js :3000)          ugc-factory (FastAPI :8000)
├── app/(app)/influencers/new/         ├── api.py  (entry point)
│   └── page.tsx  (UI principal)       ├── src/api/routes.py  (endpoints)
├── app/api/influencers/               ├── src/services/tank.py  (FAL calls)
│   ├── analyze-image/                 ├── src/agents/
│   ├── generate-image/                │   ├── agent_brown.py  (visão)
│   ├── enhance-image/                 │   ├── trinity.py  (prompts foto)
│   ├── generate-prompt-preview/       │   ├── oracle.py  (prompts vídeo)
│   ├── upload-ref/                    │   ├── cypher.py  (try-on)
│   ├── train-lora/                    │   └── refiner.py  (refinamento)
│   ├── check-lora/                    └── src/core/config.py
│   ├── generate-with-lora/
│   ├── refine-prompt/
│   ├── submit-jobs/
│   └── check-jobs/
└── .env.local
```

**Fluxo de uma geração:**
```
User clica "Gerar" (genMode="kontext")
  → page.tsx monta finalPrompt
  → POST /api/influencers/generate-image { image_url, prompt, use_kontext: true }
  → ugc-factory POST /generate-image
  → generate_image_kontext() em tank.py
  → fal-ai/flux-pro/kontext/max (edita foto in-place)
  → retorna image_url
```

---

## Variáveis de Ambiente

### `f:\influenceros\.env.local`
```
NEXT_PUBLIC_SUPABASE_URL=<sua_url_supabase>
NEXT_PUBLIC_SUPABASE_ANON_KEY=<chave_anon>
SUPABASE_SERVICE_ROLE_KEY=<service_role_key>
NEXT_PUBLIC_URL=http://localhost:3000
UGC_FACTORY_URL=http://localhost:8000
FAL_API_KEY=<fal_api_key>
ANTHROPIC_API_KEY=<anthropic_key>
```

### `f:\ugc-factory\.env`
```
ANTHROPIC_API_KEY=<anthropic_key>
TELEGRAM_BOT_TOKEN=<telegram_token>
FAL_API_KEY=<fal_api_key>
SUPABASE_URL=<supabase_url>
SUPABASE_SERVICE_KEY=<service_role_key>
HIGGSFIELD_API_KEY=
```

---

## Instalação / Setup

```powershell
# ugc-factory
cd f:\ugc-factory
python -m venv .venv
.\.venv\Scripts\pip install -e .
# ou: .\.venv\Scripts\pip install fastapi uvicorn python-multipart anthropic fal-client httpx python-dotenv pyyaml

# influenceros
cd f:\influenceros
npm install

# Iniciar ambos (em terminais separados)
cd f:\ugc-factory && .\.venv\Scripts\uvicorn.exe api:app --port 8000 --reload
cd f:\influenceros && npm run dev
```

---

## ESTADO ATUAL — Bug identificado e pendente de fix

### Problema: cabelo/pele mudam na geração (PuLID)
- **Referência:** loira, pele clara, olhos verdes
- **Gerado:** castanho escuro, pele bronzeada
- **Causa:** `genMode="kontext"` na UI chama `generate_image_pulid()` (PuLID só preserva embedding facial, não cor de cabelo/pele)
- **Fix planejado:** redirecionar "kontext" para `generate_image_kontext()` que já existe em tank.py

### Fix pendente (3 arquivos):

**1. `f:\influenceros\app\api\influencers\generate-image\route.ts` linha 24:**
```typescript
// ANTES:
const useInstantid = genMode === "instantid"
body: { use_identity_lock: !useInstantid, use_instantid: useInstantid, ... }

// DEPOIS:
const useInstantid = genMode === "instantid"
const useKontext   = genMode === "kontext"
body: { use_kontext: useKontext, use_identity_lock: !useInstantid && !useKontext, use_instantid: useInstantid, ... }
```

**2. `f:\ugc-factory\src\api\routes.py` — GenerateImageRequest:**
```python
# Adicionar campo:
use_kontext: bool = False

# Adicionar branch em api_generate_image (antes de use_identity_lock):
elif body.use_kontext:
    urls = await generate_image_kontext(body.image_url, body.prompt, body.num_images)
    return {"image_urls": urls}
```

**3. `f:\influenceros\app\(app)\influencers\new\page.tsx` linha ~452:**
```typescript
// Para genMode="kontext": prompt simplificado (Kontext preserva aparência automaticamente)
} else if (hasRef && genMode === "kontext") {
  finalPrompt = [modText || `casual UGC moment, ${arquetipo}`, "natural lighting, 9:16"].join(", ")
  finalNegative = "studio lighting, ring light, CGI, watermark"
}
// Label UI linha ~1166: "PuLID" → "Kontext", adicionar botão "PuLID" para genMode="pulid"
```

---

## Arquivo: `f:\ugc-factory\api.py`

```python
"""UGC Factory FastAPI — roda em paralelo com o bot Telegram."""

from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from src.api.routes import router

app = FastAPI(title="UGC Factory API", version="1.0.0")

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],
    allow_methods=["*"],
    allow_headers=["*"],
)

app.include_router(router)
```

---

## Arquivo: `f:\ugc-factory\src\core\config.py`

```python
import os
from dotenv import load_dotenv

load_dotenv()


class Settings:
    ANTHROPIC_API_KEY: str = os.environ["ANTHROPIC_API_KEY"]
    TELEGRAM_BOT_TOKEN: str = os.environ["TELEGRAM_BOT_TOKEN"]
    FAL_API_KEY: str = os.environ["FAL_API_KEY"]
    SUPABASE_URL: str = os.environ["SUPABASE_URL"]
    SUPABASE_SERVICE_KEY: str = os.environ["SUPABASE_SERVICE_KEY"]
    HIGGSFIELD_API_KEY: str = os.environ.get("HIGGSFIELD_API_KEY", "")

    FAL_SEEDREAM_URL = "https://queue.fal.run/fal-ai/bytedance/seedream/v4/edit"
    FAL_PULID_URL = "https://queue.fal.run/fal-ai/flux-pulid"
    FAL_KONTEXT_URL = "https://queue.fal.run/fal-ai/flux-pro/kontext/max"
    FAL_INSTANTID_URL = "https://queue.fal.run/fal-ai/instant-id"
    FAL_VEO3_URL = "https://queue.fal.run/fal-ai/veo3/fast/image-to-video"
    FAL_KLING_MOTION_URL = "https://queue.fal.run/fal-ai/kling-video/v3/pro/motion-control"

    CLAUDE_MODEL = "claude-sonnet-4-6"
    POLL_INTERVAL_SECONDS = 10


settings = Settings()
```

---

## Arquivo: `f:\ugc-factory\src\api\routes.py`

```python
"""UGC Factory HTTP API — expõe os agentes Python para o Next.js."""

import base64
import json
import os
import tempfile
import anthropic
import httpx
from fastapi import APIRouter, UploadFile, File, HTTPException
from pydantic import BaseModel
from typing import Optional

from src.core.config import settings
from src.agents.agent_brown import analyze_image
from src.agents.trinity import create_image_prompt
from src.agents.oracle import create_video_prompt, create_pov_prompt
from src.agents.cypher import generate_tryon
from src.agents.refiner import refine_image_prompt
from src.services.tank import (
    generate_image, generate_image_pulid, generate_image_kontext, generate_image_instantid,
    generate_video, upload_file, enhance_image,
    submit_lora_training, check_lora_training, generate_with_lora,
)

router = APIRouter()


def _client() -> anthropic.AsyncAnthropic:
    return anthropic.AsyncAnthropic(api_key=settings.ANTHROPIC_API_KEY)


class UploadRefRequest(BaseModel):
    image_base64: str
    mime_type: str = "image/jpeg"


class PromptRequest(BaseModel):
    instruction: str
    image_analysis: str
    influencer_lock: Optional[str] = None
    product_context: Optional[str] = None


class GenerateImageRequest(BaseModel):
    image_url: str
    prompt: str
    negative_prompt: Optional[str] = None
    use_identity_lock: bool = False
    use_instantid: bool = False
    num_images: int = 1


class RefinePromptRequest(BaseModel):
    reference_analysis: str
    generated_image_url: str
    user_feedback: Optional[str] = None
    current_prompt: Optional[str] = None
    influencer_lock: Optional[str] = None


class TryonRequest(BaseModel):
    model_image_url: str
    garment_image_url: str


class TrainLoraRequest(BaseModel):
    image_urls: list[str]
    trigger_word: str
    steps: int = 1000


class CheckLoraRequest(BaseModel):
    status_url: str


class GenerateWithLoraRequest(BaseModel):
    lora_url: str
    trigger_word: str
    prompt: str
    negative_prompt: Optional[str] = None
    num_images: int = 1


class EnhanceImageRequest(BaseModel):
    image_url: str
    creativity: float = 0.35
    resemblance: float = 0.85


class GenerateVideoRequest(BaseModel):
    image_url: str
    instruction: str
    image_analysis: str
    influencer_lock: Optional[str] = None
    product_context: Optional[str] = None
    video_style: Optional[str] = "influencer"


@router.post("/upload-ref")
async def api_upload_ref(body: UploadRefRequest):
    try:
        img_bytes = base64.b64decode(body.image_base64)
        ext = "jpg" if "jpeg" in body.mime_type else body.mime_type.split("/")[-1]
        with tempfile.NamedTemporaryFile(suffix=f".{ext}", delete=False) as f:
            f.write(img_bytes)
            tmp_path = f.name
        try:
            url = await upload_file(tmp_path)
            return {"url": url}
        finally:
            os.unlink(tmp_path)
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))


@router.post("/analyze-image")
async def api_analyze_image(file: UploadFile = File(...)):
    image_bytes = await file.read()
    try:
        raw = await analyze_image(_client(), image_bytes)
        analysis = json.loads(raw)
        return {"analysis": analysis}
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))


@router.post("/generate-prompt")
async def api_generate_prompt(body: PromptRequest):
    try:
        result = await create_image_prompt(
            _client(),
            body.instruction,
            body.image_analysis,
            body.influencer_lock or "",
            body.product_context or "",
        )
        return result
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))


@router.post("/generate-image")
async def api_generate_image(body: GenerateImageRequest):
    try:
        if body.use_instantid:
            urls = await generate_image_instantid(body.image_url, body.prompt, body.negative_prompt or "", num_images=body.num_images)
            return {"image_urls": urls}
        elif body.use_identity_lock:
            urls = await generate_image_pulid(body.image_url, body.prompt, body.negative_prompt or "", num_images=body.num_images)
            return {"image_urls": urls}
        else:
            url = await generate_image(body.image_url, body.prompt, body.negative_prompt or "")
            return {"image_url": url}
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))


@router.post("/tryon")
async def api_tryon(body: TryonRequest):
    try:
        url = await generate_tryon(body.model_image_url, body.garment_image_url)
        return {"image_url": url}
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))


@router.post("/submit-lora")
async def api_submit_lora(body: TrainLoraRequest):
    if len(body.image_urls) < 3:
        raise HTTPException(status_code=400, detail="Mínimo de 3 imagens para treinamento")
    try:
        job = await submit_lora_training(body.image_urls, body.trigger_word, body.steps)
        return job
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))


@router.post("/check-lora")
async def api_check_lora(body: CheckLoraRequest):
    try:
        return await check_lora_training(body.status_url)
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))


@router.post("/generate-with-lora")
async def api_generate_with_lora(body: GenerateWithLoraRequest):
    try:
        urls = await generate_with_lora(
            body.lora_url, body.trigger_word, body.prompt,
            body.negative_prompt or "", body.num_images,
        )
        return {"image_urls": urls}
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))


@router.post("/enhance-image")
async def api_enhance_image(body: EnhanceImageRequest):
    try:
        url = await enhance_image(body.image_url, body.creativity, body.resemblance)
        return {"image_url": url}
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))


@router.post("/generate-video")
async def api_generate_video(body: GenerateVideoRequest):
    try:
        if body.video_style == "pov":
            result = await create_pov_prompt(
                _client(), body.instruction, body.image_analysis, body.product_context or ""
            )
        else:
            result = await create_video_prompt(
                _client(),
                body.instruction,
                body.image_analysis,
                body.influencer_lock or "",
                body.product_context or "",
            )
        video_url = await generate_video(body.image_url, result["video_prompt"])
        return {"video_url": video_url}
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))


@router.post("/refine-prompt")
async def api_refine_prompt(body: RefinePromptRequest):
    try:
        async with httpx.AsyncClient(timeout=60) as client:
            resp = await client.get(body.generated_image_url)
            resp.raise_for_status()
            generated_bytes = resp.content

        generated_raw = await analyze_image(_client(), generated_bytes)

        result = await refine_image_prompt(
            _client(),
            reference_analysis=body.reference_analysis,
            generated_analysis=generated_raw,
            user_feedback=body.user_feedback or "",
            current_prompt=body.current_prompt or "",
            influencer_lock=body.influencer_lock or "",
        )
        return {
            "refined_prompt": result.get("refined_prompt", ""),
            "negative_prompt": result.get("negative_prompt", ""),
            "corrections": result.get("corrections", ""),
        }
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
```

> **NOTA para o próximo Claude:** O arquivo acima é a versão atual (sem `use_kontext`). O fix planejado adiciona `use_kontext: bool = False` em `GenerateImageRequest` e um novo branch `elif body.use_kontext` antes de `use_identity_lock`.

---

## Arquivo: `f:\ugc-factory\src\services\tank.py`

```python
"""Tank — Executor Fal.ai. Gerencia todas as chamadas de geração de mídia."""

import asyncio
import io
import os
import subprocess
import tempfile
import zipfile

import fal_client
import httpx
from src.core.config import settings

_HEADERS = {"Authorization": f"Key {settings.FAL_API_KEY}"}


def _ensure_fal_key() -> None:
    os.environ.setdefault("FAL_KEY", settings.FAL_API_KEY)


async def upload_file(file_path: str) -> str:
    _ensure_fal_key()
    loop = asyncio.get_event_loop()
    return await loop.run_in_executor(None, fal_client.upload_file, file_path)


async def _poll_until_done(client: httpx.AsyncClient, status_url: str) -> str:
    while True:
        resp = await client.get(status_url, headers=_HEADERS)
        resp.raise_for_status()
        data = resp.json()
        if data.get("status") == "COMPLETED":
            return data["response_url"]
        await asyncio.sleep(settings.POLL_INTERVAL_SECONDS)


async def generate_image_kontext(image_url: str, prompt: str, num_images: int = 1) -> list[str]:
    """Flux Kontext — edita a imagem de referência preservando identidade. Retorna lista de URLs."""
    payload: dict = {
        "image_url": image_url,
        "prompt": prompt,
        "guidance_scale": 3.5,
        "num_images": num_images,
        "aspect_ratio": "9:16",
        "output_format": "jpeg",
    }
    async with httpx.AsyncClient(timeout=300) as client:
        resp = await client.post(settings.FAL_KONTEXT_URL, headers=_HEADERS, json=payload)
        resp.raise_for_status()
        data = resp.json()
        response_url = await _poll_until_done(client, data["status_url"])
        result = await client.get(response_url, headers=_HEADERS)
        result.raise_for_status()
        return [img["url"] for img in result.json()["images"]]


async def generate_image_instantid(face_image_url: str, prompt: str, negative_prompt: str = "", num_images: int = 1) -> list[str]:
    payload: dict = {
        "face_image_url": face_image_url,
        "prompt": prompt,
        "negative_prompt": negative_prompt or "studio lighting, ring light, retouched skin, CGI, 3D render, smoothed skin, beauty filter, plastic skin, identity drift, face distortion, watermark, different person",
        "ip_adapter_scale": 0.8,
        "controlnet_conditioning_scale": 0.8,
        "num_inference_steps": 30,
        "guidance_scale": 5.0,
        "num_images": num_images,
        "image_size": {"width": 576, "height": 1024},
    }
    async with httpx.AsyncClient(timeout=300) as client:
        resp = await client.post(settings.FAL_INSTANTID_URL, headers=_HEADERS, json=payload)
        resp.raise_for_status()
        data = resp.json()
        response_url = await _poll_until_done(client, data["status_url"])
        result = await client.get(response_url, headers=_HEADERS)
        result.raise_for_status()
        return [img["url"] for img in result.json()["images"]]


async def generate_image_pulid(image_url: str, prompt: str, negative_prompt: str = "", id_scale: float = 0.8, num_images: int = 1) -> list[str]:
    payload: dict = {
        "reference_image_url": image_url,
        "prompt": prompt,
        "negative_prompt": negative_prompt or "beauty filter, heavy makeup, plastic skin, artificial eyes, CGI, 3D render, illustration, digital art, different person, identity drift, watermark, oversaturated eyes, unrealistic skin",
        "guidance_scale": 4.0,
        "num_inference_steps": 20,
        "id_weight": id_scale,
        "true_cfg": 1.0,
        "start_step": 4,
        "image_size": {"width": 576, "height": 1024},
        "num_images": num_images,
    }
    async with httpx.AsyncClient(timeout=300) as client:
        resp = await client.post(settings.FAL_PULID_URL, headers=_HEADERS, json=payload)
        resp.raise_for_status()
        data = resp.json()
        response_url = await _poll_until_done(client, data["status_url"])
        result = await client.get(response_url, headers=_HEADERS)
        result.raise_for_status()
        return [img["url"] for img in result.json().get("images", [])]


async def generate_image(image_url: str, prompt: str, negative_prompt: str = "") -> str:
    payload: dict = {"prompt": prompt, "image_urls": [image_url]}
    if negative_prompt:
        payload["negative_prompt"] = negative_prompt
    async with httpx.AsyncClient(timeout=180) as client:
        resp = await client.post(settings.FAL_SEEDREAM_URL, headers=_HEADERS, json=payload)
        resp.raise_for_status()
        data = resp.json()
        response_url = await _poll_until_done(client, data["status_url"])
        result = await client.get(response_url, headers=_HEADERS)
        result.raise_for_status()
        return result.json()["images"][0]["url"]


async def generate_video(frame_url: str, prompt: str) -> str:
    async with httpx.AsyncClient(timeout=600) as client:
        resp = await client.post(
            settings.FAL_VEO3_URL,
            headers=_HEADERS,
            json={"prompt": prompt, "image_url": frame_url, "duration": "8s", "generate_audio": True, "aspect_ratio": "9:16"},
        )
        resp.raise_for_status()
        data = resp.json()
        response_url = await _poll_until_done(client, data["status_url"])
        result = await client.get(response_url, headers=_HEADERS)
        result.raise_for_status()
        return result.json()["video"]["url"]


async def enhance_image(image_url: str, creativity: float = 0.35, resemblance: float = 0.85) -> str:
    """Clarity Upscaler 2x — adiciona textura de pele real, remove aparência plástica de IA."""
    payload = {
        "image_url": image_url,
        "upscale_factor": 2,
        "creativity": creativity,
        "resemblance": resemblance,
        "detail": 1.0,
        "prompt": "ultra-realistic skin texture, natural pores, photographic detail, candid photo",
        "negative_prompt": "smooth plastic skin, beauty filter, CGI, artificial, oversaturated, AI generated",
        "num_inference_steps": 18,
    }
    async with httpx.AsyncClient(timeout=180) as client:
        resp = await client.post("https://queue.fal.run/fal-ai/clarity-upscaler", headers=_HEADERS, json=payload)
        resp.raise_for_status()
        data = resp.json()
        response_url = await _poll_until_done(client, data["status_url"])
        result = await client.get(response_url, headers=_HEADERS)
        result.raise_for_status()
        return result.json()["image"]["url"]


async def submit_lora_training(image_urls: list[str], trigger_word: str, steps: int = 1000) -> dict:
    zip_buffer = io.BytesIO()
    async with httpx.AsyncClient(timeout=120) as http:
        with zipfile.ZipFile(zip_buffer, "w", zipfile.ZIP_DEFLATED) as zf:
            for i, url in enumerate(image_urls):
                r = await http.get(url)
                r.raise_for_status()
                zf.writestr(f"image_{i:03d}.jpg", r.content)
    zip_buffer.seek(0)
    with tempfile.NamedTemporaryFile(suffix=".zip", delete=False) as f:
        f.write(zip_buffer.read())
        tmp_zip = f.name
    try:
        zip_url = await upload_file(tmp_zip)
    finally:
        os.unlink(tmp_zip)
    payload = {
        "images_data_url": zip_url,
        "trigger_phrase": trigger_word,
        "steps": steps,
        "learning_rate": 0.00009,
        "multiresolution_training": True,
        "subject_crop": True,
        "create_masks": True,
    }
    async with httpx.AsyncClient(timeout=60) as client:
        resp = await client.post("https://queue.fal.run/fal-ai/flux-lora-portrait-trainer", headers=_HEADERS, json=payload)
        resp.raise_for_status()
        data = resp.json()
        return {"request_id": data.get("request_id"), "status_url": data.get("status_url"), "response_url": data.get("response_url")}


async def check_lora_training(status_url: str) -> dict:
    async with httpx.AsyncClient(timeout=30) as client:
        resp = await client.get(status_url, headers=_HEADERS)
        resp.raise_for_status()
        data = resp.json()
        status = data.get("status", "IN_PROGRESS")
        if status == "COMPLETED":
            result_resp = await client.get(data["response_url"], headers=_HEADERS)
            result_resp.raise_for_status()
            lora_url = result_resp.json().get("diffusers_lora_file", {}).get("url")
            return {"status": "COMPLETED", "lora_url": lora_url}
        if status in ("FAILED", "ERROR"):
            return {"status": "FAILED", "error": data.get("error", "Training failed")}
        return {"status": status}


async def generate_with_lora(lora_url: str, trigger_word: str, prompt: str, negative_prompt: str = "", num_images: int = 1) -> list[str]:
    full_prompt = f"{trigger_word}, {prompt}" if trigger_word and trigger_word.lower() not in prompt.lower() else prompt
    payload = {
        "prompt": full_prompt,
        "negative_prompt": negative_prompt or "different person, identity drift, deformed, ugly, blurry, low quality",
        "loras": [{"path": lora_url, "scale": 1.0}],
        "num_images": num_images,
        "guidance_scale": 3.5,
        "num_inference_steps": 28,
        "image_size": {"width": 576, "height": 1024},
        "enable_safety_checker": False,
    }
    async with httpx.AsyncClient(timeout=300) as client:
        resp = await client.post("https://queue.fal.run/fal-ai/flux-lora", headers=_HEADERS, json=payload)
        resp.raise_for_status()
        data = resp.json()
        response_url = await _poll_until_done(client, data["status_url"])
        result = await client.get(response_url, headers=_HEADERS)
        result.raise_for_status()
        return [img["url"] for img in result.json().get("images", [])]
```

---

## Arquivos: Next.js API Routes

### `f:\influenceros\app\api\influencers\generate-image\route.ts`
```typescript
import { NextRequest, NextResponse } from "next/server"
import { createClient } from "@/lib/supabase/server"

export const maxDuration = 120

const UGC_URL = process.env.UGC_FACTORY_URL ?? "http://localhost:8000"
const FAL_KEY = process.env.FAL_API_KEY!
const FLUX_URL = "https://fal.run/fal-ai/flux/dev"

export async function POST(req: NextRequest) {
  const supabase = await createClient()
  const { data: { user } } = await supabase.auth.getUser()
  if (!user) return NextResponse.json({ error: "Unauthorized" }, { status: 401 })

  const { prompt, negativePrompt, imageUrls, imageSize, numInferenceSteps, numImages, genMode } = await req.json()
  if (!prompt?.trim()) return NextResponse.json({ error: "prompt obrigatório" }, { status: 400 })

  const count = Math.min(Number(numImages) || 1, 4)
  const steps = Math.min(Number(numInferenceSteps) || 28, 50)

  if (imageUrls && Array.isArray(imageUrls) && imageUrls.length > 0) {
    try {
      const useInstantid = genMode === "instantid"
      const res = await fetch(`${UGC_URL}/generate-image`, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          image_url: imageUrls[0],
          prompt: prompt.trim(),
          negative_prompt: negativePrompt || "",
          use_identity_lock: !useInstantid,
          use_instantid: useInstantid,
          num_images: count,
        }),
      })
      const d = await res.json()
      if (!res.ok) throw new Error(d.detail ?? "Erro na geração")
      const urls: string[] = d.image_urls ?? (d.image_url ? [d.image_url] : [])
      if (!urls.length) throw new Error("Sem imagem na resposta")
      return NextResponse.json({ imageUrls: urls })
    } catch (err: any) {
      console.error("[generate-image] kontext error:", err?.message)
      return NextResponse.json({ error: err?.message ?? "Erro na geração" }, { status: 500 })
    }
  }

  // Sem referência → flux/dev text-to-image
  const res = await fetch(FLUX_URL, {
    method: "POST",
    headers: { "Authorization": `Key ${FAL_KEY}`, "Content-Type": "application/json" },
    body: JSON.stringify({
      prompt: prompt.trim(),
      image_size: imageSize ?? "portrait_16_9",
      num_inference_steps: steps,
      guidance_scale: 3.5,
      num_images: count,
      enable_safety_checker: true,
    }),
  })
  if (!res.ok) {
    const err = await res.text()
    return NextResponse.json({ error: "Erro na geração" }, { status: 500 })
  }
  const data = await res.json()
  const resultUrls = (data.images ?? []).map((img: { url: string }) => img.url).filter(Boolean)
  if (!resultUrls.length) return NextResponse.json({ error: "Sem imagem na resposta" }, { status: 500 })
  return NextResponse.json({ imageUrls: resultUrls })
}
```

### `f:\influenceros\app\api\influencers\enhance-image\route.ts`
```typescript
import { NextRequest, NextResponse } from "next/server"
import { createClient } from "@/lib/supabase/server"

export const maxDuration = 120

const UGC_URL = process.env.UGC_FACTORY_URL ?? "http://localhost:8000"

export async function POST(req: NextRequest) {
  const supabase = await createClient()
  const { data: { user } } = await supabase.auth.getUser()
  if (!user) return NextResponse.json({ error: "Unauthorized" }, { status: 401 })

  const { imageUrl, creativity, resemblance } = await req.json()
  if (!imageUrl) return NextResponse.json({ error: "imageUrl obrigatório" }, { status: 400 })

  const res = await fetch(`${UGC_URL}/enhance-image`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      image_url: imageUrl,
      creativity: creativity ?? 0.35,
      resemblance: resemblance ?? 0.85,
    }),
  })
  if (!res.ok) {
    const err = await res.text()
    console.error("[enhance-image] error:", err)
    return NextResponse.json({ error: "Falha ao aprimorar imagem" }, { status: 500 })
  }
  const data = await res.json()
  return NextResponse.json({ imageUrl: data.image_url })
}
```

### `f:\influenceros\app\api\influencers\upload-ref\route.ts`
```typescript
import { NextRequest, NextResponse } from "next/server"
import { createClient } from "@/lib/supabase/server"

const UGC_URL = process.env.UGC_FACTORY_URL ?? "http://localhost:8000"

export async function POST(req: NextRequest) {
  const supabase = await createClient()
  const { data: { user } } = await supabase.auth.getUser()
  if (!user) return NextResponse.json({ error: "Unauthorized" }, { status: 401 })

  const { imageBase64, mimeType = "image/jpeg" } = await req.json()
  if (!imageBase64) return NextResponse.json({ error: "imageBase64 obrigatório" }, { status: 400 })

  const res = await fetch(`${UGC_URL}/upload-ref`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ image_base64: imageBase64, mime_type: mimeType }),
  })
  if (!res.ok) {
    const err = await res.text()
    return NextResponse.json({ error: "Falha ao enviar imagem" }, { status: 500 })
  }
  const { url } = await res.json()
  return NextResponse.json({ url })
}
```

### `f:\influenceros\app\api\influencers\train-lora\route.ts`
```typescript
import { NextRequest, NextResponse } from "next/server"
import { createClient } from "@/lib/supabase/server"

export const maxDuration = 60

const UGC_URL = process.env.UGC_FACTORY_URL ?? "http://localhost:8000"

export async function POST(req: NextRequest) {
  const supabase = await createClient()
  const { data: { user } } = await supabase.auth.getUser()
  if (!user) return NextResponse.json({ error: "Unauthorized" }, { status: 401 })

  const { imageUrls, triggerWord, steps } = await req.json()
  if (!imageUrls?.length || imageUrls.length < 3)
    return NextResponse.json({ error: "Mínimo 3 imagens para treinamento" }, { status: 400 })
  if (!triggerWord?.trim())
    return NextResponse.json({ error: "triggerWord obrigatório" }, { status: 400 })

  const res = await fetch(`${UGC_URL}/submit-lora`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ image_urls: imageUrls, trigger_word: triggerWord.trim(), steps: steps ?? 2500 }),
  })
  if (!res.ok) {
    const err = await res.text()
    return NextResponse.json({ error: "Falha ao iniciar treinamento" }, { status: 500 })
  }
  return NextResponse.json(await res.json())
}
```

### `f:\influenceros\app\api\influencers\check-lora\route.ts`
```typescript
import { NextRequest, NextResponse } from "next/server"
import { createClient } from "@/lib/supabase/server"

const UGC_URL = process.env.UGC_FACTORY_URL ?? "http://localhost:8000"

export async function POST(req: NextRequest) {
  const supabase = await createClient()
  const { data: { user } } = await supabase.auth.getUser()
  if (!user) return NextResponse.json({ error: "Unauthorized" }, { status: 401 })

  const { statusUrl } = await req.json()
  if (!statusUrl) return NextResponse.json({ error: "statusUrl obrigatório" }, { status: 400 })

  const res = await fetch(`${UGC_URL}/check-lora`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ status_url: statusUrl }),
  })
  if (!res.ok) return NextResponse.json({ error: "Falha ao verificar treinamento" }, { status: 500 })
  return NextResponse.json(await res.json())
}
```

### `f:\influenceros\app\api\influencers\generate-with-lora\route.ts`
```typescript
import { NextRequest, NextResponse } from "next/server"
import { createClient } from "@/lib/supabase/server"

export const maxDuration = 120

const UGC_URL = process.env.UGC_FACTORY_URL ?? "http://localhost:8000"

export async function POST(req: NextRequest) {
  const supabase = await createClient()
  const { data: { user } } = await supabase.auth.getUser()
  if (!user) return NextResponse.json({ error: "Unauthorized" }, { status: 401 })

  const { loraUrl, triggerWord, prompt, negativePrompt, numImages } = await req.json()
  if (!loraUrl || !triggerWord || !prompt?.trim())
    return NextResponse.json({ error: "loraUrl, triggerWord e prompt são obrigatórios" }, { status: 400 })

  const res = await fetch(`${UGC_URL}/generate-with-lora`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      lora_url: loraUrl, trigger_word: triggerWord, prompt: prompt.trim(),
      negative_prompt: negativePrompt || "", num_images: Math.min(Number(numImages) || 1, 4),
    }),
  })
  if (!res.ok) return NextResponse.json({ error: "Falha na geração com LoRA" }, { status: 500 })
  const data = await res.json()
  return NextResponse.json({ imageUrls: data.image_urls ?? [] })
}
```

### `f:\influenceros\app\api\influencers\refine-prompt\route.ts`
```typescript
import { NextRequest, NextResponse } from "next/server"
import { createClient } from "@/lib/supabase/server"

export const maxDuration = 120

const UGC_URL = process.env.UGC_FACTORY_URL ?? "http://localhost:8000"

export async function POST(req: NextRequest) {
  const supabase = await createClient()
  const { data: { user } } = await supabase.auth.getUser()
  if (!user) return NextResponse.json({ error: "Unauthorized" }, { status: 401 })

  const { referenceAnalysis, generatedImageUrl, userFeedback, currentPrompt, influencerLock } = await req.json()
  if (!referenceAnalysis || !generatedImageUrl)
    return NextResponse.json({ error: "referenceAnalysis e generatedImageUrl são obrigatórios" }, { status: 400 })

  const res = await fetch(`${UGC_URL}/refine-prompt`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      reference_analysis: referenceAnalysis, generated_image_url: generatedImageUrl,
      user_feedback: userFeedback || "", current_prompt: currentPrompt || "", influencer_lock: influencerLock || "",
    }),
  })
  if (!res.ok) return NextResponse.json({ error: "Falha ao refinar prompt" }, { status: 500 })
  return NextResponse.json(await res.json())
}
```

---

## Modo de Geração — Tabela de Referência

| `genMode` | Flag enviada | Função Python | Comportamento |
|-----------|-------------|---------------|---------------|
| `"kontext"` (padrão) | `use_identity_lock: true` | `generate_image_pulid()` | ⚠️ BUG: devia ser Kontext |
| `"instantid"` | `use_instantid: true` | `generate_image_instantid()` | OK |
| `loraUrl` presente | — | `generate_with_lora()` | OK |
| sem referência | — | flux/dev text-to-image (direto no Next.js) | OK |

**Fix pendente:** `genMode="kontext"` deve chamar `generate_image_kontext()` (edita in-place).

---

## pyproject.toml

```toml
[project]
name = "ugc-factory"
version = "0.1.0"
requires-python = ">=3.11"
dependencies = [
    "anthropic>=0.49.0",
    "fal-client>=1.0.0",
    "httpx>=0.27.0",
    "python-dotenv>=1.0.0",
    "pyyaml>=6.0.1",
    "fastapi>=0.115.0",
    "uvicorn>=0.30.0",
    "python-multipart>=0.0.9",
]
```

---

## Pendências / Próximos Passos

- [ ] **FIX CRÍTICO:** Aplicar o fix do `use_kontext` nos 3 arquivos (routes.py, generate-image/route.ts, page.tsx)
- [ ] **Supabase SQL:** `ALTER TABLE influencers ADD COLUMN lora_url text, ADD COLUMN lora_trigger text, ADD COLUMN lora_status text DEFAULT 'none';`
- [ ] **Persistir LoRA:** quando check-lora retorna COMPLETED, salvar `lora_url` na row do influencer (hoje só fica em React state, se recarregar a página perde)
- [ ] **Testar Option A vs B:** gerar com LoRA (2500 steps) → depois clicar "B" (Clarity Upscaler) → comparar resultado e timing
- [ ] **Amateur Photography LoRA:** adicionar como stack opcional no `generate_with_lora` para resultado mais natural

---

---

## Arquivo: `f:\influenceros\app\(app)\influencers\new\page.tsx`
> ⚠️ Arquivo mais importante — UI principal de geração. Inclui todos os estados, handleGenerate, handleEnhance, LoRA training, timing badges.

```tsx
"use client"

import { useState } from "react"
import { useRouter } from "next/navigation"
import { createClient } from "@/lib/supabase/client"
import { ArrowLeft, ArrowRight, Check, X, ImageIcon, Sparkles, Loader2, Brain, ChevronUp, Plus, Upload } from "lucide-react"
import Link from "next/link"
import { toast } from "sonner"

const STEPS = ["Referências", "Identidade", "Físico", "Prompts", "Modos", "Revisar"]
const ARQUETIPOS = ["A Sofisticada", "A Natural", "A Vibrante", "A Misteriosa", "A Minimalista", "A Boho"]
const ASPECT_RATIOS = [
  { label: "1:1",  value: "square_hd" },
  { label: "3:4",  value: "portrait_4_3" },
  { label: "9:16", value: "portrait_16_9" },
  { label: "16:9", value: "landscape_16_9" },
  { label: "4:3",  value: "landscape_4_3" },
]

function slugify(str: string) {
  return str.toLowerCase().normalize("NFD").replace(/[̀-ͯ]/g, "").replace(/\s+/g, "-").replace(/[^a-z0-9-]/g, "")
}

export default function NewInfluencerPage() {
  const router = useRouter()
  const [step, setStep] = useState(0)
  const [saving, setSaving] = useState(false)

  const [faceFile, setFaceFile] = useState<File | null>(null)
  const [facePreview, setFacePreview] = useState<string | null>(null)
  const [faceUploadedUrl, setFaceUploadedUrl] = useState<string | null>(null)
  const [uploading, setUploading] = useState(false)
  const [analyzing, setAnalyzing] = useState(false)
  const [analysisData, setAnalysisData] = useState<any>(null)
  const [baseDnaPrompt, setBaseDnaPrompt] = useState("")
  const [modifications, setModifications] = useState("")
  const [generatedImages, setGeneratedImages] = useState<string[]>([])
  const [approvedImage, setApprovedImage] = useState<string | null>(null)
  const [generating, setGenerating] = useState(false)
  const [genProgress, setGenProgress] = useState("")
  const [extraRefs, setExtraRefs] = useState<string[]>([])
  const [aspectRatio, setAspectRatio] = useState("portrait_16_9")
  const [qualitySteps, setQualitySteps] = useState(28)
  const [numImages, setNumImages] = useState(4)
  const [selectedImage, setSelectedImage] = useState<string | null>(null)
  const [openMenu, setOpenMenu] = useState<"ratio" | "quality" | null>(null)
  const [genMode, setGenMode] = useState<"kontext" | "instantid">("kontext")
  const [lastUsedPrompt, setLastUsedPrompt] = useState("")
  const [refining, setRefining] = useState(false)
  const [showFeedback, setShowFeedback] = useState(false)
  const [refineFeedback, setRefineFeedback] = useState("")
  const [showPicker, setShowPicker] = useState(false)
  const [recentImages, setRecentImages] = useState<{url: string, prompt: string | null}[]>([])
  const [loadingPicker, setLoadingPicker] = useState(false)
  const [uploadingGallery, setUploadingGallery] = useState(false)
  const [dragOver, setDragOver] = useState(false)

  /* LoRA Training */
  const [trainingPhotos, setTrainingPhotos] = useState<{preview: string, url: string}[]>([])
  const [uploadingTraining, setUploadingTraining] = useState(false)
  const [loraStatus, setLoraStatus] = useState<"idle" | "uploading" | "training" | "ready" | "failed">("idle")
  const [loraUrl, setLoraUrl] = useState<string | null>(null)
  const [loraTrigger, setLoraTrigger] = useState("")
  const [loraStatusUrl, setLoraStatusUrl] = useState<string | null>(null)
  const [loraProgressMsg, setLoraProgressMsg] = useState("")
  const [genTiming, setGenTiming] = useState<Record<string, { duration: number, method: string }>>({})
  const [enhancing, setEnhancing] = useState<Set<string>>(new Set())

  /* Step 1: Identidade */
  const [nome, setNome] = useState("")
  const [idade, setIdade] = useState("")
  const [nacionalidade, setNacionalidade] = useState("Brasileira")
  const [arquetipo, setArquetipo] = useState(ARQUETIPOS[0])

  /* Step 2: Físico */
  const [cabeloCor, setCabeloCor] = useState("")
  const [cabeloHex, setCabeloHex] = useState("#D4B870")
  const [cabeloTipo, setCabeloTipo] = useState("liso")
  const [cabeloComp, setCabeloComp] = useState("longo")
  const [olhosCor, setOlhosCor] = useState("")
  const [olhosHex, setOlhosHex] = useState("#87CEEB")
  const [peleTom, setPeleTom] = useState("")
  const [peleHex, setPeleHex] = useState("#F5DEB3")
  const [labiosTom, setLabiosTom] = useState("")
  const [labiosHex, setLabiosHex] = useState("#C87070")
  const [sobrancelhasCor, setSobrancelhasCor] = useState("")
  const [sobrancelhasHex, setSobrancelhasHex] = useState("#A07040")

  /* Step 3: Prompts */
  const [promptCore, setPromptCore] = useState("")
  const [promptNegative, setPromptNegative] = useState("")
  const [promptPele, setPromptPele] = useState("")
  const [generatingPrompts, setGeneratingPrompts] = useState(false)

  /* Step 4: Modos */
  const [shopEstetica, setShopEstetica] = useState("")
  const [normalEstetica, setNormalEstetica] = useState("")

  async function uploadRefImage(file: File) {
    setUploading(true)
    try {
      const base64 = await new Promise<string>((resolve, reject) => {
        const reader = new FileReader()
        reader.onload = (e) => resolve((e.target?.result as string).split(",")[1])
        reader.onerror = reject
        reader.readAsDataURL(file)
      })
      const res = await fetch("/api/influencers/upload-ref", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ imageBase64: base64, mimeType: file.type || "image/jpeg" }),
      })
      if (res.ok) {
        const { url } = await res.json()
        setFaceUploadedUrl(url)
      } else {
        const err = await res.json().catch(() => ({}))
        toast.error(err?.error ?? "Erro ao enviar referência — geração sem âncora visual")
      }
    } catch {
      toast.error("Erro ao enviar referência — geração continua sem âncora visual")
    } finally {
      setUploading(false)
    }
  }

  async function openPicker() {
    setShowPicker(true)
    setLoadingPicker(true)
    const supabase = createClient()
    const { data } = await (supabase.from("influencer_images") as any)
      .select("url, prompt")
      .order("created_at", { ascending: false })
      .limit(24)
    setRecentImages(data ?? [])
    setLoadingPicker(false)
  }

  async function saveImageToHistory(url: string, prompt?: string, source = "generated") {
    try {
      const supabase = createClient()
      await (supabase.from("influencer_images") as any).insert({ url, prompt: prompt || null, source })
    } catch { }
  }

  async function handleGalleryUpload(file: File) {
    setUploadingGallery(true)
    setShowPicker(false)
    try {
      const localUrl = await new Promise<string>(resolve => {
        const reader = new FileReader()
        reader.onload = e => resolve(e.target?.result as string)
        reader.readAsDataURL(file)
      })
      setGeneratedImages(prev => [localUrl, ...prev])
      setSelectedImage(localUrl)
      await Promise.all([uploadRefImage(file), analyzeFaceImage(file)])
      await saveImageToHistory(localUrl, undefined, "uploaded")
    } catch {
      toast.error("Erro ao processar imagem")
    } finally {
      setUploadingGallery(false)
    }
  }

  async function addTrainingPhoto(file: File) {
    setUploadingTraining(true)
    try {
      const preview = await new Promise<string>(resolve => {
        const reader = new FileReader()
        reader.onload = e => resolve(e.target?.result as string)
        reader.readAsDataURL(file)
      })
      const base64 = preview.split(",")[1]
      const uploadRes = await fetch("/api/influencers/upload-ref", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ imageBase64: base64, mimeType: file.type || "image/jpeg" }),
      })
      if (!uploadRes.ok) throw new Error("Falha no upload")
      const { url } = await uploadRes.json()
      setTrainingPhotos(prev => [...prev, { preview, url }])
    } catch {
      toast.error("Erro ao adicionar foto de treinamento")
    } finally {
      setUploadingTraining(false)
    }
  }

  async function startLoraTraining() {
    if (trainingPhotos.length < 3) {
      toast.error("Adicione pelo menos 3 fotos para treinar a IA")
      return
    }
    const trigger = slugify(nome || "influencer") + "infl"
    setLoraTrigger(trigger)
    setLoraStatus("training")
    setLoraProgressMsg("Iniciando treinamento...")
    try {
      const urls = trainingPhotos.map(p => p.url)
      const res = await fetch("/api/influencers/train-lora", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ imageUrls: urls, triggerWord: trigger, steps: 2500 }),
      })
      if (!res.ok) throw new Error("Falha ao iniciar treinamento")
      const { status_url } = await res.json()
      setLoraStatusUrl(status_url)
      setLoraProgressMsg("Treinando... ~15 min")
      pollLoraStatus(status_url)
    } catch (err: any) {
      setLoraStatus("failed")
      toast.error(err?.message ?? "Erro ao iniciar treinamento")
    }
  }

  async function pollLoraStatus(statusUrl: string) {
    for (let i = 0; i < 60; i++) {
      await new Promise(r => setTimeout(r, 30000))
      try {
        const res = await fetch("/api/influencers/check-lora", {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({ statusUrl }),
        })
        if (!res.ok) continue
        const data = await res.json()
        if (data.status === "COMPLETED" && data.lora_url) {
          setLoraUrl(data.lora_url)
          setLoraStatus("ready")
          setLoraProgressMsg("")
          toast.success("✅ IA Treinada! Gerações agora com rosto consistente.")
          return
        }
        if (data.status === "FAILED") {
          setLoraStatus("failed")
          toast.error("Treinamento falhou — usando PuLID como fallback")
          return
        }
        setLoraProgressMsg(`Treinando... ${Math.round(((i + 1) / 60) * 100)}%`)
      } catch { }
    }
    setLoraStatus("failed")
    toast.error("Timeout no treinamento — usando PuLID como fallback")
  }

  async function analyzeFaceImage(file: File) {
    setAnalyzing(true)
    try {
      const base64 = await new Promise<string>((resolve, reject) => {
        const reader = new FileReader()
        reader.onload = (e) => resolve((e.target?.result as string).split(",")[1])
        reader.onerror = reject
        reader.readAsDataURL(file)
      })
      const res = await fetch("/api/influencers/analyze-image", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ imageBase64: base64, mimeType: file.type || "image/jpeg" }),
      })
      if (!res.ok) throw new Error("Falha na análise")
      const { analysis } = await res.json()
      setAnalysisData(analysis)

      const person = analysis.subject?.person ?? {}
      const meta   = analysis.meta ?? {}
      const light  = analysis.global_context?.lighting ?? {}
      const notes  = analysis.reconstruction_notes ?? {}

      const extractHex = (s: string) => { const m = s?.match(/#[0-9A-Fa-f]{6}/i); return m?.[0] ?? "" }
      const firstWord  = (s: string) => s?.split(/,|#/)[0].trim() ?? ""

      const hairStr = typeof person.hair === "string" ? person.hair : ""
      if (hairStr) {
        setCabeloCor(firstWord(hairStr))
        const hx = extractHex(hairStr); if (hx) setCabeloHex(hx)
        if (/wavy|ondulado/i.test(hairStr)) setCabeloTipo("ondulado")
        else if (/curly|cacheado|crespo/i.test(hairStr)) setCabeloTipo("cacheado")
        else if (/straight|liso/i.test(hairStr)) setCabeloTipo("liso")
        if (/long|longo/i.test(hairStr)) setCabeloComp("longo")
        else if (/medium|médio|medio/i.test(hairStr)) setCabeloComp("médio")
        else if (/short|curto/i.test(hairStr)) setCabeloComp("curto")
      }

      const eyesStr = typeof person.eyes === "string" ? person.eyes : ""
      if (eyesStr) {
        setOlhosCor(firstWord(eyesStr))
        const ex = extractHex(eyesStr); if (ex) setOlhosHex(ex)
      }

      if (person.skin_hex) setPeleHex(person.skin_hex)
      if (person.skin_description) setPeleTom(firstWord(person.skin_description))

      const skinPart = [person.skin_description, person.skin_hex].filter(Boolean).join(" ")
      const microPart = typeof person.micro_details === "string" ? person.micro_details : ""
      const posePart  = typeof person.pose === "string" ? person.pose : "looking at camera, natural expression"
      const lensNote  = typeof meta.lens_type === "string" ? meta.lens_type : "wide-angle smartphone ~24mm"
      const lightNote = typeof light.color_temperature === "string" ? `${light.color_temperature} natural light` : "natural soft light"
      const mandatoryPart = typeof notes.mandatory_elements === "string" ? notes.mandatory_elements
        : Array.isArray(notes.mandatory_elements) ? notes.mandatory_elements.join(", ") : ""

      const prompt = [
        "Photorealistic UGC photo of a woman",
        skinPart ? `${skinPart} skin` : "",
        hairStr ? `${hairStr} hair` : "",
        eyesStr ? `${eyesStr} eyes` : "",
        microPart,
        posePart,
        mandatoryPart,
        `${lensNote}, ${lightNote}`,
        "ultra-realistic authentic smartphone photo, visible pores, natural skin texture, NOT smoothed, NOT CGI, NOT beauty filter",
      ].filter(Boolean).join(", ")
      setBaseDnaPrompt(prompt)
    } catch {
    } finally {
      setAnalyzing(false)
    }
  }

  async function submitAndPoll(
    prompt: string, negativePrompt: string, imageUrls: string[],
    steps: number, count: number, mode: string,
    onProgress?: (msg: string) => void,
  ): Promise<string[]> {
    onProgress?.("Submetendo ao FAL...")
    const submitRes = await fetch("/api/influencers/submit-jobs", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ prompt, negativePrompt, imageUrls, numInferenceSteps: steps, numImages: count, genMode: mode }),
    })
    if (!submitRes.ok) {
      const errText = await submitRes.text().catch(() => "")
      throw new Error(`Submit falhou (${submitRes.status}): ${errText.slice(0, 200)}`)
    }
    const submitData = await submitRes.json()
    if (!Array.isArray(submitData?.jobs)) throw new Error("Submit: resposta inválida do servidor")

    let pendingJobs = [...submitData.jobs]
    const completedUrls: string[] = []
    const lastStatuses: string[] = []

    for (let i = 0; i < 60 && pendingJobs.length > 0; i++) {
      onProgress?.(`Aguardando FAL... (${i + 1}/60)`)
      await new Promise(r => setTimeout(r, 5000))
      const checkRes = await fetch("/api/influencers/check-jobs", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ jobs: pendingJobs }),
      })
      if (!checkRes.ok) {
        const errText = await checkRes.text().catch(() => "")
        throw new Error(`Check falhou (${checkRes.status}): ${errText.slice(0, 200)}`)
      }
      const checkData = await checkRes.json().catch(() => null)
      if (!checkData?.results) throw new Error("Check: resposta sem results")

      const { results } = checkData
      const stillPending: typeof pendingJobs = []
      const rawStatuses: string[] = []

      for (let j = 0; j < results.length; j++) {
        const r = results[j]
        lastStatuses[j] = r.falStatus ?? r.status
        rawStatuses.push(r.falStatus ?? r.status)
        if (r.status === "COMPLETED" && r.url) {
          completedUrls.push(r.url)
        } else if (r.status === "FAILED") {
          throw new Error(`Job ${j + 1} falhou no FAL`)
        } else if (r.status === "COMPLETED" && !r.url) {
          throw new Error(`Job ${j + 1} completado sem imagem`)
        } else {
          stillPending.push(pendingJobs[j])
        }
      }
      pendingJobs = stillPending
      if (rawStatuses.length > 0) onProgress?.(`FAL: ${rawStatuses[0]} (${i + 1}/60)`)
    }

    if (completedUrls.length === 0)
      throw new Error(`Timeout (5min) — status FAL: ${lastStatuses.join(", ") || "sem resposta"}`)
    return completedUrls
  }

  async function handleGenerate() {
    if (generating || analyzing) return
    const hasSomething = baseDnaPrompt || modifications.trim()
    if (!hasSomething) return

    setGenerating(true)
    const genStartTime = Date.now()
    try {
      let finalPrompt = baseDnaPrompt
        ? modifications.trim() ? `${baseDnaPrompt}, ${modifications.trim()}` : baseDnaPrompt
        : modifications.trim()
      let finalNegative = promptNegative || "studio lighting, ring light, retouched skin, CGI, 3D render, smoothed skin, AI glow, beauty filter, plastic skin, identity drift, face distortion, watermark"

      if (analysisData) {
        const modText = modifications.trim()
        const hasRef = !!faceUploadedUrl

        if (hasRef && genMode === "instantid") {
          const instruction = `Create a UGC scene for this influencer. BLOCK 1: Write only "The influencer" — do NOT describe hair, eyes, skin, or any physical feature (the face is locked by the reference image). Archetype: ${arquetipo}.${modText ? ` Scene: ${modText}.` : ""} Build blocks 2-7 for a natural action, environment, camera angle, lighting, style, and quality. Ultra-realistic UGC photo 9:16.`
          const promptRes = await fetch("/api/influencers/generate-prompt-preview", {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({
              inf: { nome: nome || "Influencer", idade: parseInt(idade) || null, nacionalidade, arquetipo },
              caract: { cabelo_cor_base: cabeloCor, cabelo_hex_base: cabeloHex, cabelo_tipo: cabeloTipo, cabelo_comprimento: cabeloComp, olhos_cor: olhosCor, olhos_hex: olhosHex, pele_tom: peleTom, pele_hex: peleHex, labios_tom: labiosTom, labios_hex: labiosHex, sobrancelhas_cor: sobrancelhasCor, sobrancelhas_hex: sobrancelhasHex },
              perfil: analysisData,
              instruction,
            }),
          })
          if (promptRes.ok) {
            const { prompts } = await promptRes.json()
            if (prompts.prompt_core) finalPrompt = prompts.prompt_core
            if (prompts.prompt_negative) finalNegative = prompts.prompt_negative
          }
        } else if (hasRef) {
          const sceneDesc = modText || `${arquetipo} influencer in a natural authentic UGC moment`
          const person = analysisData?.subject?.person ?? {}
          const notes = analysisData?.reconstruction_notes ?? {}
          const mandatory = typeof notes.mandatory_elements === "string" ? notes.mandatory_elements
            : Array.isArray(notes.mandatory_elements) ? (notes.mandatory_elements as string[]).join(", ") : ""
          const microDetails = typeof person.micro_details === "string" ? person.micro_details : ""
          const eyesStr = typeof person.eyes === "string" ? person.eyes : olhosCor
          const hairStr = typeof person.hair === "string" ? person.hair : `${cabeloCor}, ${cabeloTipo}, ${cabeloComp}`

          finalPrompt = [
            `RAW photo, candid UGC selfie of the exact same woman from the reference image.`,
            `Hair: ${hairStr}. Eyes: ${eyesStr}. Skin: ${peleTom || "natural"}, visible pores, natural texture.`,
            microDetails ? `Distinctive features: ${microDetails}.` : "",
            mandatory ? `Must preserve: ${mandatory}.` : "",
            `Scene: ${sceneDesc}.`,
            "Shot on iPhone, natural lighting, no filter, no retouching, 9:16 vertical.",
          ].filter(Boolean).join(" ")
          finalNegative = "different person, identity drift, face change, different nose, different jaw, different eye shape, altered features, hair color change, eye color change, beauty filter, heavy makeup, plastic skin, oversaturated eyes, artificial eyes, CGI, 3D render, illustration, digital art, anime, painting, studio lighting, ring light, watermark, logo, text"
        } else {
          const instruction = `Create a consistent portrait photo of the influencer in archetype ${arquetipo}. Ultra-realistic UGC style. Preserve all identity locks.${modifications.trim() ? ` Apply: ${modifications.trim()}.` : ""}`
          const promptRes = await fetch("/api/influencers/generate-prompt-preview", {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({
              inf: { nome: nome || "Influencer", idade: parseInt(idade) || null, nacionalidade, arquetipo },
              caract: { cabelo_cor_base: cabeloCor, cabelo_hex_base: cabeloHex, cabelo_tipo: cabeloTipo, cabelo_comprimento: cabeloComp, olhos_cor: olhosCor, olhos_hex: olhosHex, pele_tom: peleTom, pele_hex: peleHex, labios_tom: labiosTom, labios_hex: labiosHex, sobrancelhas_cor: sobrancelhasCor, sobrancelhas_hex: sobrancelhasHex },
              perfil: analysisData,
              instruction,
            }),
          })
          if (promptRes.ok) {
            const { prompts } = await promptRes.json()
            if (prompts.prompt_core) finalPrompt = prompts.prompt_core
            if (prompts.prompt_negative) finalNegative = prompts.prompt_negative
          }
        }
      }

      let resultUrls: string[]

      if (loraUrl && loraTrigger) {
        setGenProgress("Gerando com LoRA (rosto consistente)...")
        const sceneDesc = modifications.trim() || `${arquetipo} — natural UGC moment`
        const loraPrompt = `${sceneDesc}, ultra-realistic UGC smartphone photo 9:16, natural lighting, NOT retouched`
        const loraRes = await fetch("/api/influencers/generate-with-lora", {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({ loraUrl, triggerWord: loraTrigger, prompt: loraPrompt, negativePrompt: finalNegative, numImages }),
        })
        if (!loraRes.ok) {
          const errData = await loraRes.json().catch(() => ({}))
          throw new Error(errData?.error ?? "Falha na geração com LoRA")
        }
        resultUrls = (await loraRes.json()).imageUrls ?? []
      } else {
        setGenProgress(faceUploadedUrl ? "Gerando com PuLID (face-lock)..." : "Gerando...")
        const genRes = await fetch("/api/influencers/generate-image", {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({ prompt: finalPrompt, negativePrompt: finalNegative, imageUrls: faceUploadedUrl ? [faceUploadedUrl] : undefined, imageSize: aspectRatio, numInferenceSteps: qualitySteps, numImages, genMode }),
        })
        if (!genRes.ok) {
          const errData = await genRes.json().catch(() => ({}))
          throw new Error(errData?.error ?? "Falha na geração")
        }
        resultUrls = (await genRes.json()).imageUrls ?? []
      }

      const genDuration = Math.round((Date.now() - genStartTime) / 1000)
      const genMethod = loraUrl ? "LoRA" : (genMode === "instantid" ? "InstantID" : "PuLID")
      setGenTiming(prev => {
        const next = { ...prev }
        resultUrls.forEach(url => { next[url] = { duration: genDuration, method: genMethod } })
        return next
      })
      setGeneratedImages(prev => [...prev, ...resultUrls])
      if (resultUrls.length > 0) setSelectedImage(resultUrls[resultUrls.length - 1])
      setLastUsedPrompt(finalPrompt)
      setShowFeedback(false)
      setRefineFeedback("")
      resultUrls.forEach(url => saveImageToHistory(url, finalPrompt))
    } catch (err: any) {
      console.error("[handleGenerate]", err)
      toast.error(err?.message ?? "Erro ao gerar imagem — verifique a conexão e tente novamente.", { duration: 15000 })
    } finally {
      setGenerating(false)
      setGenProgress("")
    }
  }

  async function handleEnhance(imageUrl: string) {
    setEnhancing(prev => new Set([...prev, imageUrl]))
    const startTime = Date.now()
    try {
      const res = await fetch("/api/influencers/enhance-image", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ imageUrl }),
      })
      if (!res.ok) throw new Error("Falha ao aprimorar imagem")
      const { imageUrl: enhancedUrl } = await res.json()
      const duration = Math.round((Date.now() - startTime) / 1000)
      setGenTiming(prev => ({ ...prev, [enhancedUrl]: { duration, method: "Clarity✨" } }))
      setGeneratedImages(prev => {
        const idx = prev.indexOf(imageUrl)
        if (idx === -1) return [...prev, enhancedUrl]
        const next = [...prev]
        next.splice(idx + 1, 0, enhancedUrl)
        return next
      })
      toast.success("Imagem aprimorada!")
    } catch (err: any) {
      toast.error(err?.message ?? "Erro ao aprimorar")
    } finally {
      setEnhancing(prev => { const s = new Set(prev); s.delete(imageUrl); return s })
    }
  }

  async function handleRefine() {
    const targetImage = selectedImage ?? generatedImages[generatedImages.length - 1]
    if (!targetImage || !analysisData || !faceUploadedUrl) return
    setRefining(true)
    try {
      const refineRes = await fetch("/api/influencers/refine-prompt", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          referenceAnalysis: JSON.stringify(analysisData),
          generatedImageUrl: targetImage,
          userFeedback: refineFeedback.trim() || "",
          currentPrompt: lastUsedPrompt,
          influencerLock: [
            cabeloCor && cabeloHex ? `Hair: ${cabeloCor} ${cabeloHex}` : "",
            olhosCor && olhosHex ? `Eyes: ${olhosCor} ${olhosHex}` : "",
            peleTom && peleHex ? `Skin: ${peleTom} ${peleHex}` : "",
          ].filter(Boolean).join(". "),
        }),
      })
      if (!refineRes.ok) throw new Error("Falha ao refinar")
      const { refined_prompt, negative_prompt, corrections } = await refineRes.json()
      if (corrections) toast.success(`Correções: ${corrections}`)
      const refineUrls = await submitAndPoll(refined_prompt || lastUsedPrompt, negative_prompt || "", [faceUploadedUrl], qualitySteps, numImages, genMode)
      setGeneratedImages(prev => [...prev, ...refineUrls])
      if (refineUrls.length > 0) setSelectedImage(refineUrls[refineUrls.length - 1])
      setLastUsedPrompt(refined_prompt || lastUsedPrompt)
      setShowFeedback(false)
      setRefineFeedback("")
    } catch (err: any) {
      toast.error(err?.message ?? "Erro ao refinar imagem.")
    } finally {
      setRefining(false)
    }
  }

  async function handleGeneratePrompts() {
    if (!cabeloCor || !olhosCor || !peleTom) { toast.error("Preencha as características físicas antes de gerar os prompts."); return }
    setGeneratingPrompts(true)
    try {
      const res = await fetch("/api/influencers/generate-prompt-preview", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          inf: { nome: nome || "Influencer", idade: parseInt(idade) || null, nacionalidade, arquetipo },
          caract: { cabelo_cor_base: cabeloCor, cabelo_hex_base: cabeloHex, cabelo_tipo: cabeloTipo, cabelo_comprimento: cabeloComp, olhos_cor: olhosCor, olhos_hex: olhosHex, pele_tom: peleTom, pele_hex: peleHex, labios_tom: labiosTom, labios_hex: labiosHex, sobrancelhas_cor: sobrancelhasCor, sobrancelhas_hex: sobrancelhasHex },
          perfil: analysisData ?? {},
        }),
      })
      if (!res.ok) throw new Error("Falha na geração")
      const { prompts } = await res.json()
      if (prompts.prompt_core) setPromptCore(prompts.prompt_core)
      if (prompts.prompt_negative) setPromptNegative(prompts.prompt_negative)
      if (prompts.prompt_realismo_pele) setPromptPele(prompts.prompt_realismo_pele)
      toast.success("Prompts gerados com IA!")
    } catch { toast.error("Erro ao gerar prompts — tente novamente.") }
    finally { setGeneratingPrompts(false) }
  }

  function pickFile(setFile: (f: File) => void, setPreview?: (s: string) => void, onFile?: (f: File) => void) {
    const input = document.createElement("input")
    input.type = "file"
    input.accept = setPreview ? "image/*" : "audio/*"
    input.onchange = e => {
      const file = (e.target as HTMLInputElement).files?.[0]
      if (!file) return
      setFile(file)
      if (setPreview) { const reader = new FileReader(); reader.onload = ev => setPreview(ev.target?.result as string); reader.readAsDataURL(file) }
      onFile?.(file)
    }
    input.click()
  }

  async function handleSave() {
    if (!nome.trim()) { toast.error("Nome é obrigatório"); return }
    setSaving(true)
    const supabase = createClient()
    const slug = slugify(nome)
    let avatar_url: string | null = null
    if (faceFile) {
      try {
        const path = `${slug}/avatar.${faceFile.name.split(".").pop()}`
        const { data: uploadData, error: upErr } = await supabase.storage.from("influencers").upload(path, faceFile, { upsert: true })
        if (!upErr && uploadData) {
          const { data: urlData } = supabase.storage.from("influencers").getPublicUrl(path)
          avatar_url = urlData?.publicUrl ?? null
        }
      } catch { }
    }
    const { data: inf, error: infErr } = await (supabase.from("influencers") as any)
      .insert({ slug, nome, idade: parseInt(idade) || null, nacionalidade, arquetipo, status: "ativo", avatar_url })
      .select("id").single()
    if (infErr || !inf) { toast.error("Erro ao criar influencer: " + (infErr?.message ?? "")); setSaving(false); return }
    const id = inf.id
    await Promise.all([
      (supabase.from("influencer_caracteristicas_fisicas") as any).insert({ influencer_id: id, cabelo_cor_base: cabeloCor, cabelo_hex_base: cabeloHex, cabelo_tipo: cabeloTipo, cabelo_comprimento: cabeloComp, olhos_cor: olhosCor, olhos_hex: olhosHex, pele_tom: peleTom, pele_hex: peleHex, labios_tom: labiosTom, labios_hex: labiosHex, sobrancelhas_cor: sobrancelhasCor, sobrancelhas_hex: sobrancelhasHex }),
      (supabase.from("influencer_prompts") as any).insert({ influencer_id: id, prompt_core: promptCore, prompt_negative: promptNegative, prompt_realismo_pele: promptPele }),
      (supabase.from("influencer_modos") as any).insert([
        { influencer_id: id, modo: "tiktokshop", estetica: shopEstetica, mood: "editorial", temperatura_luz: "fria" },
        { influencer_id: id, modo: "postagem_normal", estetica: normalEstetica, mood: "lifestyle", temperatura_luz: "quente" },
      ]),
    ])
    toast.success("Influencer criada com sucesso!")
    router.push(`/influencers/${slug}`)
  }

  const canNext = step === 1 ? nome.trim().length > 0 : true

  return (
    <div className={step === 0 ? "flex flex-col h-dvh" : "p-8 max-w-2xl space-y-8"}>
      <div className={`flex items-center gap-3 shrink-0 ${step === 0 ? "px-6 pt-5 pb-0" : ""}`}>
        <Link href="/influencers" className="text-[#6A6670] hover:text-[#F0EDE8] transition-colors"><ArrowLeft size={18} /></Link>
        <div>
          <h1 className="text-xl font-semibold">Nova Influencer</h1>
          <p className="text-sm text-[#6A6670]">Crie o perfil e identidade visual da sua influencer IA</p>
        </div>
      </div>

      <div className={`flex items-center gap-1.5 shrink-0 ${step === 0 ? "px-6 pb-1" : ""}`}>
        {STEPS.map((s, i) => (
          <div key={s} className="flex items-center gap-1.5">
            <button onClick={() => i < step && setStep(i)} className={`flex items-center gap-1.5 group ${i <= step ? "cursor-pointer" : "cursor-default"}`}>
              <div className={`w-7 h-7 rounded-full flex items-center justify-center text-xs font-medium transition-colors ${i < step ? "bg-[#D4B870] text-[#0A0A0F]" : i === step ? "bg-[#D4B870]/15 border border-[#D4B870] text-[#D4B870]" : "bg-[#13131A] border border-[#2A2A35] text-[#6A6670]"}`}>
                {i < step ? <Check size={12} /> : i + 1}
              </div>
              <span className={`text-xs hidden sm:block ${i === step ? "text-[#F0EDE8]" : "text-[#6A6670]"}`}>{s}</span>
            </button>
            {i < STEPS.length - 1 && <div className={`h-px flex-1 min-w-[12px] transition-colors ${i < step ? "bg-[#D4B870]/40" : "bg-[#2A2A35]"}`} />}
          </div>
        ))}
      </div>

      {step === 0 && (
        <div className="flex-1 min-h-0 flex flex-col gap-3 px-4 pb-0">
          <div
            className={`relative flex-1 min-h-[380px] w-full rounded-2xl bg-[#0D0D14] border transition-colors ${dragOver ? "border-[#D4B870]/60 bg-[#D4B870]/5" : "border-[#2A2A35]"}`}
            onDragOver={e => { e.preventDefault(); setDragOver(true) }}
            onDragLeave={() => setDragOver(false)}
            onDrop={e => { e.preventDefault(); setDragOver(false); const f = e.dataTransfer.files?.[0]; if (f?.type.startsWith("image/")) handleGalleryUpload(f) }}
          >
            <div className="absolute top-3 right-3 z-50">
              <button onClick={openPicker} className="w-9 h-9 rounded-xl bg-[#1C1C26] border border-[#2A2A35] hover:border-[#D4B870]/40 text-[#6A6670] hover:text-[#D4B870] flex items-center justify-center transition-colors">
                {uploadingGallery ? <Loader2 size={14} className="animate-spin" /> : <Plus size={16} />}
              </button>
              {showPicker && (
                <>
                  <div className="fixed inset-0 z-40" onClick={() => setShowPicker(false)} />
                  <div className="absolute top-11 right-0 z-50 w-72 bg-[#1C1C26] border border-[#2A2A35] rounded-2xl overflow-hidden shadow-2xl">
                    <button onClick={() => pickFile(() => {}, () => {}, handleGalleryUpload)} className="w-full flex items-center gap-3 px-4 py-3 border-b border-[#2A2A35] text-sm text-[#A8A4A0] hover:text-[#F0EDE8] hover:bg-[#2A2A35]/40 transition-colors">
                      <Upload size={16} /> Faça upload de uma imagem
                    </button>
                    {loadingPicker ? (
                      <div className="flex justify-center py-6"><Loader2 size={20} className="text-[#D4B870] animate-spin" /></div>
                    ) : recentImages.length === 0 ? (
                      <p className="text-xs text-[#4A4A55] text-center py-6">Nenhum resultado encontrado</p>
                    ) : (
                      <div className="grid grid-cols-3 gap-2 p-3 max-h-60 overflow-y-auto">
                        {recentImages.map((img, i) => (
                          <img key={i} src={img.url} alt="" onClick={() => { setGeneratedImages(prev => [img.url, ...prev]); setSelectedImage(img.url); if (img.prompt) setBaseDnaPrompt(img.prompt); setShowPicker(false) }}
                            className="w-full aspect-[3/4] object-cover rounded-xl cursor-pointer border-2 border-transparent hover:border-[#D4B870]/60 transition-colors" />
                        ))}
                      </div>
                    )}
                  </div>
                </>
              )}
            </div>

            {facePreview || generatedImages.length > 0 || generating ? (
              <div className="flex gap-3 p-4 overflow-x-auto h-full rounded-2xl">
                {facePreview && (
                  <div className="relative shrink-0 h-full">
                    <img src={facePreview} alt="referência" className="h-full w-auto rounded-xl object-cover border-2 border-[#D4B870]/25 brightness-75" />
                    <div className="absolute top-2 left-2 bg-[#0A0A0F]/80 text-[#D4B870] text-[10px] font-bold px-2 py-0.5 rounded-full border border-[#D4B870]/20">REF</div>
                    {analyzing && <div className="absolute inset-0 bg-[#0A0A0F]/50 rounded-xl flex items-center justify-center"><Loader2 size={24} className="text-[#D4B870] animate-spin" /></div>}
                  </div>
                )}
                {facePreview && (generatedImages.length > 0 || generating) && <div className="shrink-0 w-px bg-[#2A2A35] self-stretch my-2" />}
                {facePreview && generatedImages.length === 0 && !generating && (
                  <div className="flex-1 flex items-center justify-center">
                    <p className="text-sm text-[#3A3A45]">{(baseDnaPrompt || modifications.trim()) ? "Clique em Gerar →" : "Adicione um prompt e clique em Gerar"}</p>
                  </div>
                )}
                {generatedImages.map((url, i) => (
                  <div key={i} onClick={() => setSelectedImage(url)} className="relative shrink-0 h-full group cursor-pointer">
                    <img src={url} alt={`gerada ${i + 1}`} className={`h-full w-auto rounded-xl object-cover border-2 transition-all duration-200 ${approvedImage === url ? "border-[#4CAF7A] shadow-[0_0_20px_rgba(76,175,122,0.3)]" : selectedImage === url ? "border-[#D4B870] shadow-[0_0_16px_rgba(212,184,112,0.25)]" : "border-transparent group-hover:border-[#D4B870]/20"}`} />
                    {approvedImage === url && <div className="absolute top-2 left-2 bg-[#4CAF7A] text-white text-[10px] font-bold px-2 py-0.5 rounded-full">APROVADA</div>}
                    {genTiming[url] && (
                      <div className="absolute top-2 right-2 bg-[#0A0A0F]/80 text-[10px] px-1.5 py-0.5 rounded-full border border-[#2A2A35] text-[#A8A4A0] font-mono">
                        {genTiming[url].method} · {genTiming[url].duration}s
                      </div>
                    )}
                    {enhancing.has(url) && (
                      <div className="absolute inset-0 bg-[#0A0A0F]/60 rounded-xl flex items-center justify-center">
                        <div className="flex flex-col items-center gap-2">
                          <Loader2 size={20} className="text-[#D4B870] animate-spin" />
                          <span className="text-[10px] text-[#D4B870]">Aprimorando…</span>
                        </div>
                      </div>
                    )}
                    <div className="absolute inset-x-0 bottom-0 h-24 bg-gradient-to-t from-[#0A0A0F]/90 to-transparent rounded-b-xl opacity-0 group-hover:opacity-100 transition-opacity flex items-end justify-center pb-3 gap-2">
                      <button onClick={e => { e.stopPropagation(); setApprovedImage(approvedImage === url ? null : url) }}
                        className={`px-3 py-1.5 rounded-lg text-xs font-semibold transition-colors ${approvedImage === url ? "bg-[#4CAF7A] text-white" : "bg-[#F0EDE8] text-[#0A0A0F] hover:bg-[#4CAF7A] hover:text-white"}`}>
                        {approvedImage === url ? "✓ Aprovada" : "Aprovar"}
                      </button>
                      <button onClick={e => { e.stopPropagation(); handleEnhance(url) }} disabled={enhancing.has(url)}
                        title="Option B: Clarity Upscaler — remove pele plástica"
                        className="flex items-center gap-1 px-2.5 py-1.5 rounded-lg text-xs font-semibold bg-[#1C1C26] border border-[#D4B870]/40 text-[#D4B870] hover:bg-[#D4B870]/15 transition-colors disabled:opacity-40">
                        <Sparkles size={10} />
                        {enhancing.has(url) ? "…" : "B"}
                      </button>
                      <button onClick={e => { e.stopPropagation(); setGeneratedImages(prev => prev.filter((_, j) => j !== i)); if (approvedImage === url) setApprovedImage(null) }}
                        className="p-1.5 bg-[#0A0A0F]/70 text-[#6A6670] hover:text-white rounded-lg transition-colors">
                        <X size={12} />
                      </button>
                    </div>
                  </div>
                ))}
                {generating && (
                  <div className="shrink-0 h-full w-[260px] flex flex-col items-center justify-center gap-3 bg-[#1C1C26] rounded-xl border-2 border-dashed border-[#D4B870]/25">
                    <Loader2 size={28} className="text-[#D4B870] animate-spin" />
                    <p className="text-xs text-[#D4B870] text-center px-3">{genProgress || "Gerando…"}</p>
                  </div>
                )}
              </div>
            ) : (
              <div className="flex flex-col items-center justify-center gap-5 text-[#4A4A55] h-full">
                <div className="w-20 h-20 rounded-2xl border-2 border-dashed border-[#2A2A35] flex items-center justify-center"><ImageIcon size={32} strokeWidth={1} /></div>
                <div className="text-center space-y-1.5">
                  <p className="text-sm text-[#6A6670]">Nenhuma imagem gerada ainda</p>
                  <p className="text-xs text-[#3A3A45]">Adicione uma referência + prompt, ou apenas um prompt</p>
                </div>
              </div>
            )}
          </div>

          <div className="bg-[#13131A] border border-[#2A2A35] rounded-2xl overflow-hidden">
            <div className="flex items-start gap-3 px-4 pt-3 pb-2">
              <div className="flex items-center gap-1.5 shrink-0">
                {facePreview && (
                  <div className="relative group/ref">
                    <img src={facePreview} alt="ref 1" className="w-11 h-11 rounded-xl object-cover border border-[#2A2A35]" />
                    {analyzing && <div className="absolute inset-0 bg-[#0A0A0F]/60 rounded-xl flex items-center justify-center"><Loader2 size={12} className="text-[#D4B870] animate-spin" /></div>}
                    <button onClick={() => { setFaceFile(null); setFacePreview(null); setFaceUploadedUrl(null); setAnalysisData(null); setBaseDnaPrompt(""); setExtraRefs([]) }}
                      className="absolute -top-1.5 -right-1.5 w-4 h-4 bg-[#2A2A35] hover:bg-[#3A3A45] text-[#A8A4A0] rounded-full flex items-center justify-center opacity-0 group-hover/ref:opacity-100 transition-opacity"><X size={9} /></button>
                  </div>
                )}
                {extraRefs.map((ref, i) => (
                  <div key={i} className="relative group/xref">
                    <img src={ref} alt={`ref ${i + 2}`} className="w-11 h-11 rounded-xl object-cover border border-[#2A2A35]" />
                    <button onClick={() => setExtraRefs(prev => prev.filter((_, j) => j !== i))}
                      className="absolute -top-1.5 -right-1.5 w-4 h-4 bg-[#2A2A35] hover:bg-[#3A3A45] text-[#A8A4A0] rounded-full flex items-center justify-center opacity-0 group-hover/xref:opacity-100 transition-opacity"><X size={9} /></button>
                  </div>
                ))}
                <button
                  onClick={() => {
                    if (!facePreview) { pickFile(f => setFaceFile(f), setFacePreview, (f) => { analyzeFaceImage(f); uploadRefImage(f) }) }
                    else {
                      const input = document.createElement("input"); input.type = "file"; input.accept = "image/*"
                      input.onchange = e => { const file = (e.target as HTMLInputElement).files?.[0]; if (!file) return; const reader = new FileReader(); reader.onload = ev => setExtraRefs(prev => [...prev, ev.target?.result as string]); reader.readAsDataURL(file) }
                      input.click()
                    }
                  }}
                  className="w-11 h-11 rounded-xl border-2 border-dashed border-[#2A2A35] hover:border-[#D4B870]/40 flex items-center justify-center text-[#4A4A55] hover:text-[#D4B870] transition-colors"
                >
                  {facePreview ? <span className="text-base leading-none">+</span> : <ImageIcon size={16} />}
                </button>
              </div>

              <div className="flex-1 flex flex-col gap-1.5">
                {uploading && <div className="flex items-center gap-1.5 text-[10px]" style={{ color: "#F0A830" }}><Loader2 size={9} className="animate-spin shrink-0" /><span>Enviando referência…</span></div>}
                {!uploading && baseDnaPrompt && <div className="flex items-center gap-1.5 text-[10px] text-[#4CAF7A]"><Brain size={9} className="shrink-0" /><span>{faceUploadedUrl ? "DNA + referência prontos" : "DNA extraído"}</span></div>}
                <textarea value={modifications} onChange={e => setModifications(e.target.value)}
                  placeholder={analyzing || uploading ? "Processando referência, aguarde…" : baseDnaPrompt ? "Ex: olhos verdes, cabelo curto, sorriso amplo…" : "Descreva a influencer para gerar a imagem base…"}
                  disabled={analyzing || uploading} rows={2}
                  className="flex-1 bg-transparent text-sm text-[#F0EDE8] placeholder-[#4A4A55] resize-none outline-none leading-relaxed disabled:opacity-50" />
              </div>

              <button onClick={handleGenerate} disabled={generating || analyzing || uploading || (!baseDnaPrompt && !modifications.trim())}
                className="shrink-0 self-end flex items-center gap-2 px-4 py-2 bg-[#D4B870] hover:bg-[#F0E0A0] text-[#0A0A0F] text-sm font-bold rounded-xl transition-colors disabled:opacity-30 disabled:cursor-not-allowed">
                {generating ? <Loader2 size={14} className="animate-spin" /> : <Sparkles size={14} />}
                {generating ? "…" : "Gerar"}
              </button>
            </div>

            {showFeedback && generatedImages.length > 0 && analysisData && faceUploadedUrl && (
              <div className="mx-3 mb-2 p-3 bg-[#0D0D14] border border-[#2A2A35] rounded-xl space-y-2">
                <p className="text-[10px] text-[#A8A4A0]">O agente vai analisar a referência e a imagem gerada para corrigir automaticamente. Adicione feedback opcional:</p>
                <textarea value={refineFeedback} onChange={e => setRefineFeedback(e.target.value)}
                  placeholder="Ex: pele muito bronzeada, cabelo muito escuro, olhos errados…" rows={2}
                  className="w-full bg-transparent text-xs text-[#F0EDE8] placeholder-[#4A4A55] resize-none outline-none border border-[#1C1C26] rounded-lg p-2 leading-relaxed" />
                <div className="flex items-center gap-2 justify-end">
                  <button onClick={() => setShowFeedback(false)} className="text-[10px] text-[#6A6670] hover:text-[#A8A4A0] transition-colors">Cancelar</button>
                  <button onClick={handleRefine} disabled={refining}
                    className="flex items-center gap-1.5 px-3 py-1 bg-[#D4B870] hover:bg-[#F0E0A0] text-[#0A0A0F] text-xs font-bold rounded-lg transition-colors disabled:opacity-40">
                    {refining ? <Loader2 size={10} className="animate-spin" /> : <Brain size={10} />}
                    {refining ? "Analisando…" : "Refinar com IA"}
                  </button>
                </div>
              </div>
            )}

            {openMenu && <div className="fixed inset-0 z-40" onClick={() => setOpenMenu(null)} />}
            <div className="relative flex items-center gap-3 px-3 pb-2.5 pt-1.5 border-t border-[#1C1C26]">
              <div className="relative z-50">
                <button onClick={() => setOpenMenu(openMenu === "ratio" ? null : "ratio")}
                  className="flex items-center gap-1 px-2.5 py-1 bg-[#0D0D14] border border-[#1C1C26] rounded-lg text-[11px] text-[#A8A4A0] hover:text-[#F0EDE8] hover:border-[#2A2A35] transition-colors">
                  {ASPECT_RATIOS.find(r => r.value === aspectRatio)?.label ?? "9:16"}
                  <ChevronUp size={10} className={`transition-transform duration-150 ${openMenu === "ratio" ? "" : "rotate-180"}`} />
                </button>
                {openMenu === "ratio" && (
                  <div className="absolute bottom-full mb-2 left-0 bg-[#1C1C26] border border-[#2A2A35] rounded-xl py-1 min-w-[100px] shadow-2xl">
                    {ASPECT_RATIOS.map(({ label, value }) => (
                      <button key={value} onClick={() => { setAspectRatio(value); setOpenMenu(null) }}
                        className={`w-full flex items-center gap-2 px-3 py-1.5 text-xs transition-colors ${aspectRatio === value ? "text-[#F0EDE8] bg-[#2A2A35]/60" : "text-[#6A6670] hover:text-[#A8A4A0] hover:bg-[#2A2A35]/40"}`}>
                        {aspectRatio === value ? <Check size={10} className="text-[#D4B870] shrink-0" /> : <span className="w-[10px] shrink-0" />}
                        {label}
                      </button>
                    ))}
                  </div>
                )}
              </div>

              <div className="relative z-50">
                <button onClick={() => setOpenMenu(openMenu === "quality" ? null : "quality")}
                  className="flex items-center gap-1 px-2.5 py-1 bg-[#0D0D14] border border-[#1C1C26] rounded-lg text-[11px] text-[#A8A4A0] hover:text-[#F0EDE8] hover:border-[#2A2A35] transition-colors">
                  ♡ {qualitySteps === 28 ? "1K" : "2K"}
                  <ChevronUp size={10} className={`transition-transform duration-150 ${openMenu === "quality" ? "" : "rotate-180"}`} />
                </button>
                {openMenu === "quality" && (
                  <div className="absolute bottom-full mb-2 left-0 bg-[#1C1C26] border border-[#2A2A35] rounded-xl py-1 min-w-[90px] shadow-2xl">
                    {([["1K", 28], ["2K", 50]] as [string, number][]).map(([label, steps]) => (
                      <button key={steps} onClick={() => { setQualitySteps(steps); setOpenMenu(null) }}
                        className={`w-full flex items-center gap-2 px-3 py-1.5 text-xs transition-colors ${qualitySteps === steps ? "text-[#F0EDE8] bg-[#2A2A35]/60" : "text-[#6A6670] hover:text-[#A8A4A0] hover:bg-[#2A2A35]/40"}`}>
                        {qualitySteps === steps ? <Check size={10} className="text-[#D4B870] shrink-0" /> : <span className="w-[10px] shrink-0" />}
                        {label}
                      </button>
                    ))}
                  </div>
                )}
              </div>

              <div className="flex items-center gap-0.5 text-[11px] text-[#6A6670]">
                <button onClick={() => setNumImages(n => Math.max(1, n - 1))} className="w-5 h-5 flex items-center justify-center hover:text-[#A8A4A0] transition-colors text-base leading-none">—</button>
                <span className="text-[#A8A4A0] min-w-[28px] text-center">{numImages}/4</span>
                <button onClick={() => setNumImages(n => Math.min(4, n + 1))} className="w-5 h-5 flex items-center justify-center hover:text-[#A8A4A0] transition-colors text-base leading-none">+</button>
              </div>

              {faceUploadedUrl && !loraUrl && (
                <div className="flex items-center gap-0.5 bg-[#0D0D14] border border-[#1C1C26] rounded-lg p-0.5">
                  <button onClick={() => setGenMode("instantid")} className={`px-2 py-0.5 rounded-md text-[10px] font-medium transition-colors ${genMode === "instantid" ? "bg-[#2A2A35] text-[#D4B870]" : "text-[#6A6670] hover:text-[#A8A4A0]"}`}>InstantID</button>
                  <button onClick={() => setGenMode("kontext")} className={`px-2 py-0.5 rounded-md text-[10px] font-medium transition-colors ${genMode === "kontext" ? "bg-[#2A2A35] text-[#D4B870]" : "text-[#6A6670] hover:text-[#A8A4A0]"}`}>PuLID</button>
                </div>
              )}

              {loraStatus === "ready" && <span className="flex items-center gap-1 px-2 py-0.5 bg-green-500/15 border border-green-500/30 rounded-lg text-[10px] text-green-400 font-medium"><Check size={10} /> LoRA Pronto</span>}
              {loraStatus === "training" && <span className="flex items-center gap-1 px-2 py-0.5 bg-[#D4B870]/10 border border-[#D4B870]/30 rounded-lg text-[10px] text-[#D4B870] font-medium"><Loader2 size={10} className="animate-spin" /> {loraProgressMsg || "Treinando..."}</span>}

              {loraStatus === "idle" && trainingPhotos.length >= 3 && (
                <button onClick={startLoraTraining} className="flex items-center gap-1 px-2.5 py-1 bg-[#D4B870]/10 border border-[#D4B870]/40 hover:bg-[#D4B870]/20 rounded-lg text-[10px] text-[#D4B870] font-medium transition-colors">
                  <Brain size={10} /> Treinar IA ({trainingPhotos.length} fotos)
                </button>
              )}

              <label className="flex items-center gap-1 px-2 py-0.5 bg-[#0D0D14] border border-[#1C1C26] rounded-lg text-[10px] text-[#6A6670] hover:text-[#A8A4A0] hover:border-[#2A2A35] transition-colors cursor-pointer">
                {uploadingTraining ? <Loader2 size={10} className="animate-spin" /> : <Plus size={10} />}
                Treino {trainingPhotos.length > 0 ? `(${trainingPhotos.length})` : ""}
                <input type="file" accept="image/*" className="hidden" onChange={e => { const f = e.target.files?.[0]; if (f) addTrainingPhoto(f); e.target.value = "" }} />
              </label>

              <div className="flex-1" />

              {generatedImages.length > 0 && (
                <div className="flex items-center gap-1.5">
                  {analysisData && faceUploadedUrl && (
                    <button onClick={() => setShowFeedback(v => !v)}
                      className={`flex items-center gap-1 px-2.5 py-1 rounded-lg text-xs font-medium transition-colors border ${showFeedback ? "bg-[#D4B870]/15 text-[#D4B870] border-[#D4B870]/30" : "bg-transparent text-[#A8A4A0] border-[#F0EDE8]/10 hover:border-[#D4B870]/30 hover:text-[#D4B870]"}`}>
                      <Brain size={10} /> Ajustar
                    </button>
                  )}
                  <button onClick={() => { const target = selectedImage ?? generatedImages[generatedImages.length - 1]; setApprovedImage(prev => prev === target ? null : target) }}
                    className={`flex items-center gap-1.5 px-3 py-1 rounded-lg text-xs font-semibold transition-colors border ${approvedImage ? "bg-[#4CAF7A]/15 text-[#4CAF7A] border-[#4CAF7A]/30" : "bg-transparent text-[#F0EDE8]/70 border-[#F0EDE8]/10 hover:bg-[#4CAF7A] hover:text-white hover:border-[#4CAF7A]"}`}>
                    <Check size={10} /> {approvedImage ? "Aprovada" : "Aprovar"}
                  </button>
                </div>
              )}
            </div>
          </div>
        </div>
      )}

      {step > 0 && (
        <div className="bg-[#13131A] border border-[#2A2A35] rounded-xl p-6 space-y-5">
          {step === 1 && (
            <>
              <h2 className="text-sm font-medium text-[#A8A4A0] uppercase tracking-wider">Identidade</h2>
              <Field label="Nome *"><input value={nome} onChange={e => setNome(e.target.value)} placeholder="Ex: Mikaela" className="input-base" /></Field>
              <div className="grid grid-cols-2 gap-4">
                <Field label="Idade"><input type="number" value={idade} onChange={e => setIdade(e.target.value)} placeholder="25" className="input-base" /></Field>
                <Field label="Nacionalidade"><input value={nacionalidade} onChange={e => setNacionalidade(e.target.value)} className="input-base" /></Field>
              </div>
              <Field label="Arquétipo"><select value={arquetipo} onChange={e => setArquetipo(e.target.value)} className="input-base">{ARQUETIPOS.map(a => <option key={a}>{a}</option>)}</select></Field>
              {analysisData && <div className="flex items-center gap-2 text-xs text-[#6A6670]"><Brain size={12} className="text-[#4CAF7A]" /><span>Idade e arquétipo pré-sugeridos pela análise. Confirme ou ajuste.</span></div>}
              {nome && <p className="text-xs text-[#6A6670]">Slug gerado: <span className="font-mono text-[#D4B870]">/{slugify(nome)}</span></p>}
            </>
          )}
          {step === 2 && (
            <>
              <div className="flex items-center justify-between">
                <h2 className="text-sm font-medium text-[#A8A4A0] uppercase tracking-wider">Características Físicas</h2>
                {analysisData && <span className="text-[11px] text-[#4CAF7A] flex items-center gap-1"><Brain size={11} /> Pré-preenchido pela IA</span>}
              </div>
              <ColorField label="Cabelo" cor={cabeloCor} setCor={setCabeloCor} hex={cabeloHex} setHex={setCabeloHex} />
              <div className="grid grid-cols-2 gap-4">
                <Field label="Tipo de cabelo"><select value={cabeloTipo} onChange={e => setCabeloTipo(e.target.value)} className="input-base">{["liso","ondulado","cacheado","crespo"].map(t => <option key={t}>{t}</option>)}</select></Field>
                <Field label="Comprimento"><select value={cabeloComp} onChange={e => setCabeloComp(e.target.value)} className="input-base">{["curto","médio","longo","extra longo"].map(t => <option key={t}>{t}</option>)}</select></Field>
              </div>
              <ColorField label="Olhos" cor={olhosCor} setCor={setOlhosCor} hex={olhosHex} setHex={setOlhosHex} />
              <ColorField label="Pele (tom)" cor={peleTom} setCor={setPeleTom} hex={peleHex} setHex={setPeleHex} />
              <ColorField label="Lábios" cor={labiosTom} setCor={setLabiosTom} hex={labiosHex} setHex={setLabiosHex} />
              <ColorField label="Sobrancelhas" cor={sobrancelhasCor} setCor={setSobrancelhasCor} hex={sobrancelhasHex} setHex={setSobrancelhasHex} />
            </>
          )}
          {step === 3 && (
            <>
              <div className="flex items-center justify-between">
                <h2 className="text-sm font-medium text-[#A8A4A0] uppercase tracking-wider">Prompts</h2>
                <button onClick={handleGeneratePrompts} disabled={generatingPrompts} className="flex items-center gap-1.5 px-3 py-1.5 bg-[#D4B870]/10 hover:bg-[#D4B870]/20 text-[#D4B870] text-xs font-medium rounded-lg transition-colors disabled:opacity-50">
                  {generatingPrompts ? <Loader2 size={12} className="animate-spin" /> : <Sparkles size={12} />}
                  {generatingPrompts ? "Gerando…" : "Gerar com IA"}
                </button>
              </div>
              <Field label="Core Prompt"><textarea value={promptCore} onChange={e => setPromptCore(e.target.value)} rows={4} placeholder="Descrição visual principal..." className="input-base resize-none font-mono text-xs" /></Field>
              <Field label="Negative Prompt"><textarea value={promptNegative} onChange={e => setPromptNegative(e.target.value)} rows={3} placeholder="O que evitar..." className="input-base resize-none font-mono text-xs" /></Field>
              <Field label="Prompt Realismo de Pele"><textarea value={promptPele} onChange={e => setPromptPele(e.target.value)} rows={3} placeholder="Técnica tirar cara de borracha..." className="input-base resize-none font-mono text-xs" /></Field>
            </>
          )}
          {step === 4 && (
            <>
              <h2 className="text-sm font-medium text-[#A8A4A0] uppercase tracking-wider">Modos de Postagem</h2>
              <div className="space-y-4">
                <div className="p-4 rounded-lg border border-[#D4B870]/20 bg-[#D4B870]/5 space-y-3">
                  <div className="flex items-center gap-2 mb-1"><div className="w-2 h-2 rounded-full bg-[#D4B870]" /><span className="text-sm font-medium text-[#D4B870]">TikTok Shop</span><span className="text-xs text-[#6A6670]">— Comercial / Editorial</span></div>
                  <Field label="Estética do modo"><input value={shopEstetica} onChange={e => setShopEstetica(e.target.value)} placeholder="Ex: Editorial fria, fundo neutro" className="input-base" /></Field>
                </div>
                <div className="p-4 rounded-lg border border-[#7AB8D4]/20 bg-[#7AB8D4]/5 space-y-3">
                  <div className="flex items-center gap-2 mb-1"><div className="w-2 h-2 rounded-full bg-[#7AB8D4]" /><span className="text-sm font-medium text-[#7AB8D4]">Postagem Normal</span><span className="text-xs text-[#6A6670]">— Lifestyle / Orgânico</span></div>
                  <Field label="Estética do modo"><input value={normalEstetica} onChange={e => setNormalEstetica(e.target.value)} placeholder="Ex: Lifestyle quente, luz dourada" className="input-base" /></Field>
                </div>
              </div>
            </>
          )}
          {step === 5 && (
            <div className="space-y-4">
              <h2 className="text-sm font-medium text-[#A8A4A0] uppercase tracking-wider">Revisar antes de criar</h2>
              {facePreview && (
                <div className="flex items-center gap-4 p-3 bg-[#0A0A0F] rounded-lg border border-[#2A2A35]">
                  <img src={facePreview} alt="avatar" className="w-14 h-14 rounded-full object-cover border-2 border-[#D4B870]/30" />
                  <div><p className="text-sm font-semibold">{nome || "—"}</p><p className="text-xs text-[#A8A4A0]">{arquetipo} · {idade} anos · {nacionalidade}</p></div>
                </div>
              )}
              <div className="space-y-0 divide-y divide-[#1C1C26]">
                <ReviewRow label="Nome" value={nome} />
                <ReviewRow label="Arquétipo" value={arquetipo} />
                <ReviewRow label="Idade / Nac." value={`${idade || "—"} anos · ${nacionalidade}`} />
                <ReviewRow label="Cabelo" value={cabeloCor} hex={cabeloHex} />
                <ReviewRow label="Olhos" value={olhosCor} hex={olhosHex} />
                <ReviewRow label="Pele" value={peleTom} hex={peleHex} />
                <ReviewRow label="Core Prompt" value={promptCore ? `${promptCore.slice(0, 60)}…` : "—"} />
                <ReviewRow label="Modo Shop" value={shopEstetica || "—"} />
                <ReviewRow label="Modo Normal" value={normalEstetica || "—"} />
                {analysisData && <ReviewRow label="Análise IA" value="✓ Características extraídas automaticamente" />}
              </div>
            </div>
          )}
        </div>
      )}

      <div className={`flex justify-between items-center shrink-0 ${step === 0 ? "px-6 py-3 border-t border-[#1C1C26]" : ""}`}>
        <button onClick={() => setStep(s => s - 1)} disabled={step === 0} className="flex items-center gap-2 px-4 py-2 text-sm text-[#A8A4A0] hover:text-[#F0EDE8] disabled:opacity-30 transition-colors"><ArrowLeft size={14} /> Voltar</button>
        <span className="text-xs text-[#6A6670]">{step + 1} / {STEPS.length}</span>
        {step < STEPS.length - 1 ? (
          <button onClick={() => setStep(s => s + 1)} disabled={!canNext || analyzing} className="flex items-center gap-2 px-4 py-2 bg-[#D4B870] hover:bg-[#F0E0A0] text-[#0A0A0F] text-sm font-semibold rounded-lg transition-colors disabled:opacity-50">
            {analyzing && <Loader2 size={14} className="animate-spin" />}
            {analyzing ? "Analisando…" : "Próximo"} {!analyzing && <ArrowRight size={14} />}
          </button>
        ) : (
          <button onClick={handleSave} disabled={saving} className="flex items-center gap-2 px-5 py-2 bg-[#D4B870] hover:bg-[#F0E0A0] text-[#0A0A0F] text-sm font-semibold rounded-lg transition-colors disabled:opacity-50">
            <Check size={14} /> {saving ? "Criando…" : "Criar Influencer"}
          </button>
        )}
      </div>
    </div>
  )
}

function Field({ label, children }: { label: string; children: React.ReactNode }) {
  return <div className="space-y-1.5"><label className="text-xs text-[#6A6670] uppercase tracking-wider">{label}</label>{children}</div>
}

function ColorField({ label, cor, setCor, hex, setHex }: { label: string; cor: string; setCor: (v: string) => void; hex: string; setHex: (v: string) => void }) {
  return (
    <div className="flex gap-3 items-end">
      <div className="flex-1 space-y-1.5">
        <label className="text-xs text-[#6A6670] uppercase tracking-wider">{label}</label>
        <input value={cor} onChange={e => setCor(e.target.value)} placeholder={`Cor ${label.toLowerCase()}`} className="input-base" />
      </div>
      <div className="space-y-1.5">
        <label className="text-xs text-[#6A6670] uppercase tracking-wider">HEX</label>
        <div className="flex items-center gap-2">
          <div className="w-9 h-9 rounded-md border border-[#2A2A35]" style={{ backgroundColor: hex }} />
          <input type="color" value={hex} onChange={e => setHex(e.target.value)} className="w-9 h-9 rounded-md cursor-pointer bg-transparent border-0 p-0" />
        </div>
      </div>
    </div>
  )
}

function ReviewRow({ label, value, hex }: { label: string; value: string; hex?: string }) {
  return (
    <div className="flex items-center gap-3 py-2.5">
      <span className="text-xs text-[#6A6670] w-28 shrink-0">{label}</span>
      {hex && <div className="w-4 h-4 rounded-sm border border-[#2A2A35] shrink-0" style={{ backgroundColor: hex }} />}
      <span className="text-sm text-[#A8A4A0] truncate">{value}</span>
    </div>
  )
}
```

---

*Este documento foi gerado automaticamente pelo script `update-devlog.ps1`*
