import os
from operator import itemgetter
from langchain.chains import create_sql_query_chain
from langchain_openai import ChatOpenAI
from langchain_community.utilities import SQLDatabase
from langchain_community.tools.sql_database.tool import QuerySQLDataBaseTool
from langchain.chat_models import ChatOpenAI
from langchain.schema import StrOutputParser
from langchain.schema.runnable import Runnable
from langchain.schema.runnable.config import RunnableConfig
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough
from langchain_core.prompts import PromptTemplate
import chainlit as cl
from dotenv import load_dotenv, dotenv_values
load_dotenv()
API_KEY = os.environ["OPEN_API_KEY"]
DATABASE = os.environ["DATABASE"]
DBPASS = os.environ["DBPASS"]
db = SQLDatabase.from_uri(
    f"postgresql+psycopg2://postgres:{(DBPASS)}@localhost:5432/aact",
)
llm = ChatOpenAI(temperature=0.2, openai_api_key=API_KEY, model_name='gpt-4-turbo',cache=False)
execute_query = QuerySQLDataBaseTool(db=db)
write_query = create_sql_query_chain(llm, db)
answer_prompt = PromptTemplate.from_template(
    """
Given the following user question, corresponding SQL query, and SQL result, answer the user question.
Question: {question}
SQL Query: {query}
SQL Result: {result}
Answer: """
)
@cl.on_chat_start
async def on_chat_start():
    chain = (
        RunnablePassthrough.assign(query=write_query).assign(
            result=itemgetter("query") | execute_query
        )
        | answer_prompt
        | llm
        | StrOutputParser()
    )
    cl.user_session.set("chain", chain)
@cl.on_message
async def on_message(message: cl.Message):
    runnable = cl.user_session.get("chain")  # type: Runnable
    msg = cl.Message(content="")
    async for chunk in runnable.astream(
        {"question": message.content},
        config=RunnableConfig(callbacks=[cl.LangchainCallbackHandler()]),
    ):
        await msg.stream_token(chunk)
    await msg.send()
'''
chain = (
        RunnablePassthrough.assign(query=write_query).assign(
            result=itemgetter("query") | execute_query
        )
        | answer_prompt
        | llm
        | StrOutputParser()
    )
def get_prompt():
    print("Type 'exit' to quit")
    while True:
        prompt = input("Enter a prompt: ")
        if prompt.lower() == 'exit':
            print('Exiting...')
            break
        else:
            try:
                print(chain.invoke({'question':prompt}))
            except Exception as e:
                print("error")
                print(e)
get_prompt()
'''

















