---
layout: post
title:  "JUST CODE"
date:   2023-07-01 08:49:48 +0000
categories: jekyll update
---

this is a sample text used for showing something else.

Lorem ipsum dolor sit amet, `consectetur adipiscing elit`, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Proin sed libero enim sed faucibus turpis in. Dolor morbi non arcu risus quis varius quam. In hac habitasse platea dictumst vestibulum. Sit amet facilisis magna etiam tempor orci eu lobortis elementum. Et ultrices neque ornare aenean euismod elementum nisi quis eleifend. Lacus suspendisse faucibus interdum posuere lorem. Ac tincidunt vitae semper quis lectus nulla at volutpat. Elementum tempus egestas sed sed risus. Diam donec adipiscing tristique risus. Non tellus orci ac auctor augue mauris augue neque gravida. Sollicitudin tempor id eu nisl. Pellentesque massa placerat duis ultricies lacus sed turpis. Fermentum leo vel orci porta non pulvinar neque laoreet suspendisse. Consequat mauris nunc congue nisi vitae suscipit tellus mauris. Volutpat sed cras ornare arcu dui. Quam pellentesque nec nam aliquam sem et. At tellus at urna condimentum mattis pellentesque.

Turpis massa tincidunt dui ut ornare lectus sit amet. Diam quis enim lobortis scelerisque fermentum dui faucibus in. Ipsum consequat nisl vel pretium lectus quam. Amet consectetur adipiscing elit duis tristique sollicitudin. Habitant morbi tristique senectus et netus et. Blandit turpis cursus in hac habitasse platea dictumst quisque. Ultrices in iaculis nunc sed. Ut tortor pretium viverra suspendisse potenti nullam. Sed euismod nisi porta lorem mollis aliquam ut porttitor. In tellus integer feugiat scelerisque varius morbi. Lectus quam id leo in vitae turpis massa sed elementum. Facilisis gravida neque convallis a cras semper auctor neque vitae. Amet facilisis magna etiam tempor. Ut faucibus pulvinar elementum integer enim neque volutpat ac. Arcu dui vivamus arcu felis bibendum ut tristique et egestas. Imperdiet dui accumsan sit amet nulla facilisi morbi tempus iaculis. Nunc sed velit dignissim sodales ut eu sem integer. Sit amet purus gravida quis. Morbi quis commodo odio aenean sed adipiscing diam. Enim tortor at auctor urna nunc.

Nibh mauris cursus mattis molestie. Vitae elementum curabitur vitae nunc sed velit dignissim sodales ut. Sollicitudin ac orci phasellus egestas tellus. Morbi enim nunc faucibus a pellentesque sit amet porttitor eget. Blandit turpis cursus in hac habitasse platea. Quisque non tellus orci ac auctor augue. Quam nulla porttitor massa id neque aliquam. Augue ut lectus arcu bibendum at varius vel pharetra vel. Sed euismod nisi porta lorem. Adipiscing enim eu turpis egestas pretium. Vel orci porta non pulvinar neque laoreet suspendisse interdum consectetur. Id velit ut tortor pretium viverra. Aliquet sagittis id consectetur purus ut faucibus pulvinar. Vitae nunc sed velit dignissim sodales ut eu. Facilisis magna etiam tempor orci eu lobortis elementum. Nisl purus in mollis nunc sed. Cursus euismod quis viverra nibh cras pulvinar mattis nunc sed. Donec ac odio tempor orci dapibus ultrices.

Tellus molestie nunc non blandit massa enim nec dui nunc. Gravida cum sociis natoque penatibus et magnis. Felis eget nunc lobortis mattis aliquam faucibus purus. Amet mauris commodo quis imperdiet. Sed vulputate odio ut enim blandit volutpat maecenas volutpat. Risus in hendrerit gravida rutrum quisque non tellus. Ut venenatis tellus in metus vulputate. Vel turpis nunc eget lorem dolor sed viverra ipsum. Molestie at elementum eu facilisis. Lorem ipsum dolor sit amet consectetur adipiscing. Semper auctor neque vitae tempus quam pellentesque. Adipiscing elit pellentesque habitant morbi tristique senectus et. Fermentum posuere urna nec tincidunt praesent semper feugiat nibh sed. Dolor sit amet consectetur adipiscing elit ut aliquam. Aenean pharetra magna ac placerat vestibulum lectus mauris ultrices eros. Tellus id interdum velit laoreet id donec ultrices tincidunt arcu. Felis eget velit aliquet sagittis id consectetur. Maecenas volutpat blandit aliquam etiam. Amet justo donec enim diam vulputate.

Arcu cursus euismod quis viverra. Ullamcorper eget nulla facilisi etiam. Quisque sagittis purus sit amet volutpat consequat mauris nunc congue. A cras semper auctor neque vitae tempus quam. Morbi tincidunt ornare massa eget. Vestibulum sed arcu non odio euismod lacinia at quis risus. Eu sem integer vitae justo eget. Quam adipiscing vitae proin sagittis nisl rhoncus mattis. Sodales ut etiam sit amet nisl purus in. Vel turpis nunc eget lorem dolor sed viverra. Odio tempor orci dapibus ultrices in iaculis nunc. Nunc sed velit dignissim sodales. Tortor vitae purus faucibus ornare. Massa sed elementum tempus egestas. Etiam non quam lacus suspendisse faucibus interdum posuere lorem. Interdum posuere lorem ipsum dolor sit amet consectetur adipiscing elit.

```python
from fastapi import APIRouter, Depends
from fastapi.responses import RedirectResponse
from schemas.user import UserCreate
from schemas.url import URLBase
from utils.database_utils import get_db
from utils.router_utils import generate_keys
from sqlalchemy.orm import Session
from sqlalchemy import select
from turtle_link_shortener.models import User as UserModel, URL as URLModel, UserURL
from turtle_link_shortener.security import Password
from turtle_link_shortener.errors import UserNotFound, URLNotValid, URLForwardError
from pydantic import AnyUrl
from pydantic.tools import parse_obj_as
from datetime import datetime

user = APIRouter()


@user.post("/user/create", tags=["users"])
async def create_user(new_user: UserCreate, db: Session = Depends(get_db)):
    db_user = UserModel(**new_user.dict())
    db_user.password = Password.hash(db_user.password)

    db.add(db_user)
    db.commit()
    db.refresh(db_user)

    return db_user


@user.get("/user/{user_id}", tags=["users"])
async def get_user(user_id: int, db: Session = Depends(get_db)):
    db_user = db.get(UserModel, user_id)
    if db_user is None:
        raise UserNotFound(status_code=404, detail=f"No user with id={user_id}")
    return db_user


@user.post("/user/{user_id}/shorten", tags=["users"])
async def shorten_link(
    url: URLBase, user_id: int, custom_key: str, db: Session = Depends(get_db)
):
    if not parse_obj_as(AnyUrl, url.target_url):
        raise URLNotValid(status_code=400, detail="Your provided URL is not valid")

    if db.get(UserModel, user_id) is None:
        raise UserNotFound(status_code=404, 
                detail=f"No user with id={user_id}. Cannot proceed with user_auth!")

    # TODO - also check if the custom_url has been used by another user

    # set the characters to be used for tokenization
    key, secret_key = generate_keys(custom_key)
    now = datetime.now()

    # add the generated data to URL Model Table
    db_url = URLModel(target_url=url.target_url, custom_url=key, 
                      secret_key=secret_key, time_created=now)

    db_user_url = UserURL(user_id=user_id, link_created=key, link_time_created=now)

    db.add(db_url)
    db.commit()
    db.refresh(db_url)
    
    db.add(db_user_url)
    db.commit()
    db.refresh(db_user_url)

    db_url.custom_url = key
    db_url.admin_url = secret_key

    return db_url


@user.get("/{custom_url}", tags=["users"])
async def forward(custom_url: str, db: Session = Depends(get_db)):
    url = db.scalar(select(URLModel).where(URLModel.custom_url == custom_url))

    if url is not None:
        url.clicks += 1
        db.commit()
        db.refresh(url)
        
        return RedirectResponse(url.target_url, status_code=307)
    else:
        raise URLForwardError(status_code=404, 
                detail=f"Bad Request. {custom_url} not linked to any valid url!")


@user.get("/user/{user_id}/links", tags=["users"])
async def get_links(user_id: int, db: Session = Depends(get_db)):
    db_user = db.get(UserModel, user_id)

    if db_user is None:
        raise UserNotFound(status_code=404, detail=f"No user with id={user_id}")
        
    links = db.scalars(select(UserURL).where(UserURL.user_id == db_user.id)).all()

    return [link for link in links]
    
```

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
