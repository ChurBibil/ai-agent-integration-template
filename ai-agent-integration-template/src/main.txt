import os
import asyncio
import logging
import json
from typing import List, Optional
from datetime import datetime
import google.generativeai as genai
from dotenv import load_dotenv
from pydantic import BaseModel, Field, ValidationError

# Настройка логирования уровня Enterprise
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s | %(levelname)s | %(name)s | %(message)s'
)
logger = logging.getLogger("AI-Integration-Service")

class AnalysisReport(BaseModel):
    subject: str
    executive_summary: str
    risk_assessment: int = Field(ge=1, le=10)
    strategic_steps: List[str]
    generated_at: datetime = Field(default_factory=datetime.now)

class GeminiCoreAgent:
    def __init__(self, model_name: str = "gemini-3-pro-preview"):
        load_dotenv()
        api_key = os.getenv("GEMINI_API_KEY")
        if not api_key:
            raise EnvironmentError("Critical: GEMINI_API_KEY is missing")
        
        genai.configure(api_key=api_key)
        self.model = genai.GenerativeModel(
            model_name=model_name,
            system_instruction="Ты — ведущий системный архитектор. Ответ — строго JSON по схеме AnalysisReport."
        )
        logger.info(f"Agent initiated with {model_name}")

    async def _call_api_with_backoff(self, prompt: str, max_retries: int = 3):
        for attempt in range(max_retries):
            try:
                # Таймаут на запрос к нейронке
                response = await asyncio.wait_for(
                    self.model.generate_content_async(
                        prompt,
                        generation_config=genai.GenerationConfig(
                            response_mime_type="application/json",
                            temperature=0.15 
                        )
                    ),
                    timeout=30.0
                )
                return response.text
            except Exception as e:
                if attempt == max_retries - 1: raise e
                wait = (attempt + 1) * 3
                logger.warning(f"Retry {attempt+1}/{max_retries} after {wait}s due to error: {e}")
                await asyncio.sleep(wait)

    async def run_analysis(self, query: str) -> Optional[AnalysisReport]:
        try:
            raw_data = await self._call_api_with_backoff(query)
            data = json.loads(raw_data)
            return AnalysisReport(**data)
        except (ValidationError, json.JSONDecodeError, Exception) as e:
            logger.error(f"Inference cycle failed: {e}")
            return None

async def main():
    agent = GeminiCoreAgent()
    query = "Анализ миграции корпоративной инфраструктуры на AI-управление."
    report = await agent.run_analysis(query)
    if report:
        print(report.model_dump_json(indent=4))

if __name__ == "__main__":
    asyncio.run(main())