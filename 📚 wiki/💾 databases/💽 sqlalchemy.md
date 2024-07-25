##### Создание асинхронного подключения
```python
from sqlalchemy.ext.asyncio import create_async_engine, async_sessionmaker  
from sqlalchemy.orm import declarative_base  
  
import settings  
  
engine = create_async_engine(
	f"postgresql+asyncpg://{USER}:{PASSWORD}@{HOST}:{PORT}/{NAME}",
	future=True
)  
async_session = async_sessionmaker(bind=engine, expire_on_commit=False)  
  
Base = declarative_base()
```
##### Alembic base commands
```sh
alembic init migrations
alembic revision --autogenerate -m "migration messager"
alembic upgrade heads
```
##### Basic User model 
```python
class User(Base):  
    __tablename__ = "user"  
  
    id = mapped_column(Integer, primary_key=True, autoincrement=True)  
    email = mapped_column(String(127), unique=True, nullable=False)  
    name = mapped_column(String(31), nullable=True)  
    surname = mapped_column(String(63), nullable=True)  
    role = mapped_column(Enum(Role))  
    registered_at = mapped_column(TIMESTAMP, default=datetime.now)  
  
    @validates("email")  
    def validate_email(self, key, value):  
        try:  
            email = validate_email(value, check_deliverability=False).normalized 
        except EmailNotValidError as e:  
            raise ValueError(str(e))  
        else:  
            return email
```
##### Basic models with foreign keys
