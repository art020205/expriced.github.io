# Отчет по лабораторной работе

## Тема

Разработка серверного приложения на FastAPI для организации и проведения хакатонов.

Система поддерживает два вида пользователей:

- организаторы;
- участники.

Участники могут регистрироваться на хакатоны, но перед участием их заявка должна быть подтверждена организатором.

## Ссылки на выполненные практики

- Практика 1: [commit aa0dfb7cbbc4cc94b1d25adbfa84194bf294cd3f](https://github.com/TonikX/ITMO_ICT_WebDevelopment_tools_2025-2026/commit/aa0dfb7cbbc4cc94b1d25adbfa84194bf294cd3f)
- Практика 2: [commit 4af1974eef342662eb754b25b66170445c54633d](https://github.com/TonikX/ITMO_ICT_WebDevelopment_tools_2025-2026/commit/4af1974eef342662eb754b25b66170445c54633d)
- Практика 3: [commit 8b4f69ad2b68a59bcc830815a1101e0234d7768b](https://github.com/TonikX/ITMO_ICT_WebDevelopment_tools_2025-2026/commit/8b4f69ad2b68a59bcc830815a1101e0234d7768b)
- Финальная версия лабораторной работы: [commit b0de99127809a78175335e12db9c96c7d242d05b](https://github.com/TonikX/ITMO_ICT_WebDevelopment_tools_2025-2026/commit/b0de99127809a78175335e12db9c96c7d242d05b)

## Используемые технологии

- Python
- FastAPI
- SQLModel
- SQLAlchemy
- PostgreSQL
- Alembic
- Docker Compose
- uv
- PyJWT
- python-dotenv

## Реализованные возможности

- Регистрация и авторизация пользователей.
- Генерация JWT-токенов.
- Аутентификация по JWT-токену.
- Хэширование паролей.
- Получение информации о текущем пользователе.
- Получение списка пользователей.
- Смена пароля.
- Создание и редактирование хакатонов.
- Регистрация участников на хакатон.
- Подтверждение и отклонение заявок участников организатором.
- Создание задач хакатона.
- Формирование команд.
- Связь участников и команд через ассоциативную таблицу.
- Загрузка решений командами.
- Оценка решений организатором.
- Вложенные ответы для связанных объектов.
- Миграции БД через Alembic.

## Схема базы данных

В проекте реализовано 8 таблиц:

- `user`
- `hackathon`
- `participantregistration`
- `task`
- `team`
- `teammember`
- `submission`
- `evaluation`

Связи:

- `user` 1:N `hackathon`
- `user` 1:N `participantregistration`
- `hackathon` 1:N `participantregistration`
- `hackathon` 1:N `task`
- `hackathon` 1:N `team`
- `user` M:N `team` через `teammember`
- `team` 1:N `submission`
- `task` 1:N `submission`
- `submission` 1:N `evaluation`
- `user` 1:N `evaluation`

Ассоциативная сущность `teammember` содержит дополнительные поля:

- `member_role`
- `joined_at`

## Реализованные эндпоинты

### Авторизация

| Endpoint | Методы |
|---|---|
| `/auth/register` | `POST` |
| `/auth/login` | `POST` |

### Пользователи

| Endpoint | Методы |
|---|---|
| `/users/me` | `GET` |
| `/users/me/password` | `PATCH` |
| `/users` | `GET`, `POST` |
| `/users/{user_id}` | `GET`, `PATCH`, `DELETE` |

### Хакатоны

| Endpoint | Методы |
|---|---|
| `/hackathons` | `GET`, `POST` |
| `/hackathons/{hackathon_id}` | `GET`, `PATCH`, `DELETE` |

### Регистрация участников на хакатон

| Endpoint | Методы |
|---|---|
| `/hackathons/{hackathon_id}/registrations` | `GET`, `POST` |
| `/registrations/{registration_id}/approve` | `PATCH` |
| `/registrations/{registration_id}/reject` | `PATCH` |

### Задачи

| Endpoint | Методы |
|---|---|
| `/hackathons/{hackathon_id}/tasks` | `GET`, `POST` |
| `/tasks/{task_id}` | `GET`, `PATCH`, `DELETE` |

### Команды

| Endpoint | Методы |
|---|---|
| `/hackathons/{hackathon_id}/teams` | `GET`, `POST` |
| `/teams/{team_id}` | `GET`, `PATCH`, `DELETE` |
| `/teams/{team_id}/members` | `GET`, `POST` |
| `/teams/{team_id}/members/{user_id}` | `PATCH`, `DELETE` |

### Решения

| Endpoint | Методы |
|---|---|
| `/tasks/{task_id}/submissions` | `GET`, `POST` |
| `/submissions/{submission_id}` | `GET`, `PATCH`, `DELETE` |

### Оценки

| Endpoint | Методы |
|---|---|
| `/submissions/{submission_id}/evaluations` | `GET`, `POST` |
| `/evaluations/{evaluation_id}` | `PATCH`, `DELETE` |

## Модели данных

```python
from datetime import datetime
from enum import Enum
from typing import List, Optional

from sqlmodel import Field, Relationship, SQLModel


class UserRole(str, Enum):
    organizer = "organizer"
    participant = "participant"


class HackathonStatus(str, Enum):
    planned = "planned"
    registration_open = "registration_open"
    in_progress = "in_progress"
    finished = "finished"


class RegistrationStatus(str, Enum):
    pending = "pending"
    approved = "approved"
    rejected = "rejected"


class UserBase(SQLModel):
    full_name: str
    email: str = Field(unique=True, index=True)
    phone: Optional[str] = None
    role: UserRole


class User(UserBase, table=True):
    id: Optional[int] = Field(default=None, primary_key=True)
    password_hash: str
    is_active: bool = True
    created_at: datetime = Field(default_factory=datetime.utcnow)

    organized_hackathons: List["Hackathon"] = Relationship(back_populates="organizer")
    registrations: List["ParticipantRegistration"] = Relationship(back_populates="user")
    team_links: List["TeamMember"] = Relationship(back_populates="user")
    submissions: List["Submission"] = Relationship(back_populates="uploaded_by")
    evaluations: List["Evaluation"] = Relationship(back_populates="judge")


class UserCreate(UserBase):
    password: str


class UserUpdate(SQLModel):
    full_name: Optional[str] = None
    email: Optional[str] = None
    phone: Optional[str] = None
    role: Optional[UserRole] = None
    is_active: Optional[bool] = None


class UserRead(UserBase):
    id: int
    is_active: bool
    created_at: datetime


class UserLogin(SQLModel):
    email: str
    password: str


class TokenResponse(SQLModel):
    access_token: str
    token_type: str = "bearer"


class ChangePassword(SQLModel):
    old_password: str
    new_password: str


class HackathonBase(SQLModel):
    title: str
    description: str
    status: HackathonStatus = HackathonStatus.planned
    start_date: str
    end_date: str
    organizer_id: Optional[int] = Field(default=None, foreign_key="user.id")


class Hackathon(HackathonBase, table=True):
    id: Optional[int] = Field(default=None, primary_key=True)
    created_at: datetime = Field(default_factory=datetime.utcnow)

    organizer: Optional[User] = Relationship(back_populates="organized_hackathons")
    registrations: List["ParticipantRegistration"] = Relationship(back_populates="hackathon")
    tasks: List["Task"] = Relationship(back_populates="hackathon")
    teams: List["Team"] = Relationship(back_populates="hackathon")


class HackathonCreate(HackathonBase):
    pass


class HackathonUpdate(SQLModel):
    title: Optional[str] = None
    description: Optional[str] = None
    status: Optional[HackathonStatus] = None
    start_date: Optional[str] = None
    end_date: Optional[str] = None
    organizer_id: Optional[int] = None


class HackathonRead(HackathonBase):
    id: int
    created_at: datetime


class ParticipantRegistrationBase(SQLModel):
    user_id: int = Field(foreign_key="user.id")
    hackathon_id: int = Field(foreign_key="hackathon.id")
    status: RegistrationStatus = RegistrationStatus.pending
    skills: str
    about: Optional[str] = None


class ParticipantRegistration(ParticipantRegistrationBase, table=True):
    id: Optional[int] = Field(default=None, primary_key=True)
    registered_at: datetime = Field(default_factory=datetime.utcnow)
    confirmed_at: Optional[datetime] = None

    user: Optional[User] = Relationship(back_populates="registrations")
    hackathon: Optional[Hackathon] = Relationship(back_populates="registrations")


class ParticipantRegistrationCreate(SQLModel):
    skills: str
    about: Optional[str] = None


class ParticipantRegistrationRead(ParticipantRegistrationBase):
    id: int
    registered_at: datetime
    confirmed_at: Optional[datetime] = None


class ParticipantRegistrationReadWithUser(ParticipantRegistrationRead):
    user: Optional[UserRead] = None


class TaskBase(SQLModel):
    title: str
    description: str
    requirements: str
    evaluation_criteria: str
    hackathon_id: Optional[int] = Field(default=None, foreign_key="hackathon.id")


class Task(TaskBase, table=True):
    id: Optional[int] = Field(default=None, primary_key=True)
    created_at: datetime = Field(default_factory=datetime.utcnow)

    hackathon: Optional[Hackathon] = Relationship(back_populates="tasks")
    submissions: List["Submission"] = Relationship(back_populates="task")


class TaskCreate(SQLModel):
    title: str
    description: str
    requirements: str
    evaluation_criteria: str


class TaskUpdate(SQLModel):
    title: Optional[str] = None
    description: Optional[str] = None
    requirements: Optional[str] = None
    evaluation_criteria: Optional[str] = None


class TaskRead(TaskBase):
    id: int
    created_at: datetime


class TeamBase(SQLModel):
    name: str
    description: Optional[str] = None
    hackathon_id: Optional[int] = Field(default=None, foreign_key="hackathon.id")


class Team(TeamBase, table=True):
    id: Optional[int] = Field(default=None, primary_key=True)
    created_at: datetime = Field(default_factory=datetime.utcnow)

    hackathon: Optional[Hackathon] = Relationship(back_populates="teams")
    members: List["TeamMember"] = Relationship(back_populates="team")
    submissions: List["Submission"] = Relationship(back_populates="team")


class TeamCreate(SQLModel):
    name: str
    description: Optional[str] = None


class TeamUpdate(SQLModel):
    name: Optional[str] = None
    description: Optional[str] = None


class TeamRead(TeamBase):
    id: int
    created_at: datetime


class TeamMemberBase(SQLModel):
    user_id: int = Field(foreign_key="user.id", primary_key=True)
    team_id: int = Field(foreign_key="team.id", primary_key=True)
    member_role: str


class TeamMember(TeamMemberBase, table=True):
    joined_at: datetime = Field(default_factory=datetime.utcnow)

    user: Optional[User] = Relationship(back_populates="team_links")
    team: Optional[Team] = Relationship(back_populates="members")


class TeamMemberCreate(SQLModel):
    user_id: int
    member_role: str


class TeamMemberUpdate(SQLModel):
    member_role: Optional[str] = None


class TeamMemberRead(TeamMemberBase):
    joined_at: datetime


class TeamMemberReadWithUser(TeamMemberRead):
    user: Optional[UserRead] = None


class TeamMemberReadWithTeam(TeamMemberRead):
    team: Optional[TeamRead] = None


class TeamReadWithMembers(TeamRead):
    members: List[TeamMemberReadWithUser] = Field(default_factory=list)


class UserReadWithTeams(UserRead):
    team_links: List[TeamMemberReadWithTeam] = Field(default_factory=list)


class SubmissionBase(SQLModel):
    team_id: int = Field(foreign_key="team.id")
    task_id: int = Field(foreign_key="task.id")
    uploaded_by_id: int = Field(foreign_key="user.id")
    title: str
    description: Optional[str] = None
    repository_url: Optional[str] = None
    demo_url: Optional[str] = None


class Submission(SubmissionBase, table=True):
    id: Optional[int] = Field(default=None, primary_key=True)
    created_at: datetime = Field(default_factory=datetime.utcnow)

    team: Optional[Team] = Relationship(back_populates="submissions")
    task: Optional[Task] = Relationship(back_populates="submissions")
    uploaded_by: Optional[User] = Relationship(back_populates="submissions")
    evaluations: List["Evaluation"] = Relationship(back_populates="submission")


class SubmissionCreate(SQLModel):
    team_id: int
    title: str
    description: Optional[str] = None
    repository_url: Optional[str] = None
    demo_url: Optional[str] = None


class SubmissionUpdate(SQLModel):
    title: Optional[str] = None
    description: Optional[str] = None
    repository_url: Optional[str] = None
    demo_url: Optional[str] = None


class SubmissionRead(SubmissionBase):
    id: int
    created_at: datetime


class EvaluationBase(SQLModel):
    submission_id: int = Field(foreign_key="submission.id")
    judge_id: int = Field(foreign_key="user.id")
    score: int = Field(ge=0, le=100)
    comment: Optional[str] = None


class Evaluation(EvaluationBase, table=True):
    id: Optional[int] = Field(default=None, primary_key=True)
    created_at: datetime = Field(default_factory=datetime.utcnow)

    submission: Optional[Submission] = Relationship(back_populates="evaluations")
    judge: Optional[User] = Relationship(back_populates="evaluations")


class EvaluationCreate(SQLModel):
    score: int = Field(ge=0, le=100)
    comment: Optional[str] = None


class EvaluationUpdate(SQLModel):
    score: Optional[int] = Field(default=None, ge=0, le=100)
    comment: Optional[str] = None


class EvaluationRead(EvaluationBase):
    id: int
    created_at: datetime


class SubmissionReadWithRelations(SubmissionRead):
    team: Optional[TeamRead] = None
    task: Optional[TaskRead] = None
    uploaded_by: Optional[UserRead] = None
    evaluations: List[EvaluationRead] = Field(default_factory=list)


class HackathonReadWithRelations(HackathonRead):
    organizer: Optional[UserRead] = None
    registrations: List[ParticipantRegistrationReadWithUser] = Field(default_factory=list)
    tasks: List[TaskRead] = Field(default_factory=list)
    teams: List[TeamReadWithMembers] = Field(default_factory=list)


class MessageResponse(SQLModel):
    status: int
    message: str
```

## Код соединения с базой данных

```python
import os
from collections.abc import Generator

from dotenv import load_dotenv
from sqlmodel import Session, create_engine


load_dotenv()

DATABASE_URL = os.getenv(
    "DB_ADMIN",
    "postgresql://postgres:postgres@localhost:5432/hackathon_db",
)

engine = create_engine(DATABASE_URL, echo=True)


def get_session() -> Generator[Session, None, None]:
    with Session(engine) as session:
        yield session
```

## Код авторизации и JWT

```python
import base64
import hashlib
import hmac
import os
from datetime import datetime, timedelta, timezone

import jwt
from dotenv import load_dotenv
from fastapi import HTTPException
from fastapi.security import HTTPAuthorizationCredentials
from sqlmodel import Session, select

from models import User


load_dotenv()

SECRET_KEY = os.getenv("SECRET_KEY", "dev-secret-key")
TOKEN_TTL_MINUTES = int(os.getenv("TOKEN_TTL_MINUTES", "60"))


def hash_password(password: str) -> str:
    salt = os.urandom(16)
    digest = hashlib.pbkdf2_hmac("sha256", password.encode(), salt, 100_000)
    return f"pbkdf2_sha256${base64.urlsafe_b64encode(salt).decode()}${base64.urlsafe_b64encode(digest).decode()}"


def verify_password(password: str, password_hash: str) -> bool:
    try:
        algorithm, salt_value, digest_value = password_hash.split("$")
    except ValueError:
        return False

    if algorithm != "pbkdf2_sha256":
        return False

    salt = base64.urlsafe_b64decode(salt_value.encode())
    expected_digest = base64.urlsafe_b64decode(digest_value.encode())
    actual_digest = hashlib.pbkdf2_hmac("sha256", password.encode(), salt, 100_000)
    return hmac.compare_digest(actual_digest, expected_digest)


def create_access_token(user: User) -> str:
    now = datetime.now(timezone.utc)
    payload = {
        "sub": str(user.id),
        "role": user.role.value,
        "iat": now,
        "exp": now + timedelta(minutes=TOKEN_TTL_MINUTES),
    }
    return jwt.encode(payload, SECRET_KEY, algorithm="HS256")


def decode_access_token(token: str) -> dict:
    try:
        return jwt.decode(token, SECRET_KEY, algorithms=["HS256"])
    except jwt.ExpiredSignatureError as exc:
        raise HTTPException(status_code=401, detail="Token expired") from exc
    except jwt.InvalidTokenError as exc:
        raise HTTPException(status_code=401, detail="Invalid token") from exc


def get_current_user(
    session: Session,
    credentials: HTTPAuthorizationCredentials | None,
) -> User:
    if not credentials:
        raise HTTPException(status_code=401, detail="Authorization header is missing")

    token = credentials.credentials
    if token.lower().startswith("bearer "):
        token = token[7:].strip()

    payload = decode_access_token(token)
    user_id = int(payload["sub"])
    user = session.get(User, user_id)

    if not user or not user.is_active:
        raise HTTPException(status_code=401, detail="User is inactive or does not exist")

    return user


def get_user_by_email(session: Session, email: str) -> User | None:
    return session.exec(select(User).where(User.email == email)).first()
```

## Миграции

Для миграций используется Alembic. URL базы данных передается через `.env`:

```text
DB_ADMIN=postgresql://postgres:postgres@localhost:5432/hackathon_db
```

В `alembic.ini` используется подстановка:

```ini
sqlalchemy.url = %(DB_ADMIN)s
```

В `migrations/env.py` переменная загружается из `.env` и передается в конфигурацию Alembic:

```python
database_url = os.getenv("DB_ADMIN")
if not database_url:
    raise RuntimeError("DB_ADMIN is not set. Add it to the .env file.")

config.set_main_option("DB_ADMIN", database_url)
```

## Запуск

```bash
uv sync
docker compose up -d
uv run alembic upgrade head
uv run fastapi run main.py
```

Документация API доступна по адресу:

```text
http://127.0.0.1:8000/docs
```

## Тестовые данные

Для заполнения базы данных подготовлен файл `seed.sql`.

```bash
psql -U postgres -d hackathon_db -f seed.sql
```

Пароль для всех пользователей из `seed.sql`:

```text
password123
```
