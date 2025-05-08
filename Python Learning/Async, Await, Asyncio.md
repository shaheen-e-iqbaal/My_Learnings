
-> await can only be used inside method which is declared as async def.

-> async def method can't be called from non async def method.

-> to call async def method, we have two ways. either use await or use asyncio.run(method_name()).

-> method with async def is called coroutine.