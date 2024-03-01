# Redux Toolkit

https://redux-toolkit.js.org/tutorials/quick-start

## installation

pnpm add @reduxjs/toolkit
pnpm add react-redux

# Redux Toolkit Ornegi

## `Kurulum`

```
npm install || pnpm install || yarn
```

## `Kullanilan Kutuphaneler`

-   `@reduxjs/toolkit`
-   `react-redux`
-   `axios`
-   `react-router-dom`
-   `@mui/material-ui`
-   `@emotion/react`
-   `@emotion/styled`

## `Kullanilacak API`:

-   https://newsapi.org/
    (Get Key butonuna tiklanarak key alinabilir)

-   URL:
    const url = `https://newsapi.org/v2/top-headlines?country=tr&apiKey=${API_KEY}`;

## `Kullanilacak Araclar`

-   `Redux Dev Tools` : Chrome uzerinde calisan ve global state uzerinde yapilan tum degisikliklerin takip edilmesini saglayan tarayici uzantisidir. Indirmek icin [tiklayiniz.](https://chrome.google.com/webstore/detail/redux-devtools/lmhkpmbekcpmknklioeibfkpmmfibljd?utm_source=chrome-ntp-icon)`

# Redux Toolkit

## 1-src/features/altinda ne kadar slice varsa slicelar yazilir

-   authSlice.jsx

```jsx
import { createSlice } from "@reduxjs/toolkit";

const initialState = {
    user: "",
};

//? Slice ile bir global state'in hem action type'lari , hem action creator fonksiyonlarini hem de reducer'i tek bir hamlede olusturabiliriz.

const authSlice = createSlice({
    name: "auth", //? action type'ı olusturmak icin
    initialState, //? state'in bslangic degeri icin
    reducers: {
        //? action creator fonks. ve reducer icin
        setUser: (state, action) => {
            state.user = action.payload;
        },
        clearUser: (state) => {
            state.user = "";
        },
    },
});

//? olusan action fonksiyonları sliceAdi.actions 'dan destructure edilerek export edilir.
export const { setUser, clearUser } = authSlice.actions;

//? yazilan slice'in reducer'i sliceAdi.reducer seklinde export edilmelidir.
export default authSlice.reducer;
```

-   newsSlice.jsx

```jsx
import { createSlice, createAsyncThunk } from "@reduxjs/toolkit";
import axios from "axios";

const initialState = {
    news: [],
    loading: false,
    error: false,
};

export const getNews = createAsyncThunk(
    "getNewsFunc", //? action type name

    async () => {
        const API_KEY = "1a1a999e0d7240a6bd2dead87bcca78e";
        const url = `https://newsapi.org/v2/top-headlines?country=us&apiKey=${API_KEY}`;

        const { data } = await axios(url);
        console.log(data);
        return data.articles;
    }
);

const newsSlice = createSlice({
    name: "news",
    initialState,
    reducers: {
        clearNews: (state) => {
            state.news = [];
        },
    },
    //? createAyncThunk metedo bir middleware olarak API gibi dis kaynakli isteklerin redux ortaminda olsuturulmasini saglar. Ancak API^deki durumlara gore state'lerin guncellenmesini saglamaz. Bunun icin slice icersiindeki extraReducer kismi kullanilir.

    //? API isteklerinde 3 farkli alt durum meydana gelir. Bunlar baslama (pending), basarili bitme (fullfilled) ve basariz bitme (rejected) dir.

    extraReducers: (builder) => {
        builder
            .addCase(getNews.pending, (state) => {
                state.loading = true;
            })
            .addCase(getNews.fulfilled, (state, action) => {
                state.news = action.payload;
                state.loading = false;
            })
            .addCase(getNews.rejected, (state) => {
                state.error = true;
                state.loading = false;
            });
    },
});

export const { clearNews } = newsSlice.actions;

export default newsSlice.reducer;
```

## 2- src/app/store.jsx ile store olusturulur.

-   store.jsx

```jsx
import { configureStore } from "@reduxjs/toolkit";
import authReducer from "../features/authSlice";
import newsReducer from "../features/newsSlice";

export const store = configureStore({
    reducer: {
        auth: authReducer,
        api: newsReducer,
    },
    devTools: process.env.NODE_ENV !== "production",
    //? eger gelistirme asamasi prodcution ise o zaman yukaridaki ifade false dondurur ve dolayisiyla devTool kullanima kapali olur.
});
```

## 3- olusan stor'a tüm componentlerden ulasabilmek icin App.jsx içine Provider eklenir

-   App.jxs

```jsx
import { Provider } from "react-redux";
import AppRouter from "./router/AppRouter";
import { store } from "./app/store";

function App() {
    return (
        <Provider store={store}>
            <AppRouter />
        </Provider>
    );
}

export default App;
```

## 4- store daki state'i doldurmak ve ihtiyac olan componentlerde kullanmak için useDispatch() hooku ve useSelector() hooku kullanılır

-   Login.jsx (useDispatch() ile statei doldurmak icin)

```jsx
import * as React from "react";
import Button from "@mui/material/Button";
import TextField from "@mui/material/TextField";
import Link from "@mui/material/Link";
import Box from "@mui/material/Box";
import Avatar from "@mui/material/Avatar";
import Typography from "@mui/material/Typography";
import Container from "@mui/material/Container";
import { useState } from "react";
import { useDispatch } from "react-redux";
import { setUser } from "../features/authSlice";
import { useNavigate } from "react-router-dom";

export default function Login() {
    const [email, setEmail] = useState("");
    const [password, setPassword] = useState("");
    const dispatch = useDispatch();
    const navigate = useNavigate();

    const handleSubmit = (e) => {
        e.preventDefault();
        //? kullanici bilgileri ile state'i guncelleme istegi
        dispatch(setUser({ email, password }));
        setEmail("");
        setPassword("");
        navigate("/");
    };

    return (
        <Container component="main" maxWidth="xs">
            <Box
                sx={{
                    marginTop: 8,
                    display: "flex",
                    flexDirection: "column",
                    alignItems: "center",
                }}
            >
                <Avatar
                    alt="avatar_img"
                    src="https://cdn.pixabay.com/photo/2017/03/21/02/00/user-2160923_960_720.png"
                    sx={{ width: 100, height: 100 }}
                />
                <Typography component="h1" variant="h5">
                    Sign in
                </Typography>
                <Box component="form" onSubmit={handleSubmit} sx={{ mt: 1 }}>
                    <TextField
                        margin="normal"
                        required
                        fullWidth
                        id="email"
                        label="Email Address"
                        name="email"
                        autoComplete="email"
                        autoFocus
                        value={email}
                        onChange={(e) => setEmail(e.target.value)}
                    />
                    <TextField
                        margin="normal"
                        required
                        fullWidth
                        name="password"
                        label="Password"
                        type="password"
                        id="password"
                        autoComplete="current-password"
                        value={password}
                        onChange={(e) => setPassword(e.target.value)}
                    />

                    <Button
                        type="submit"
                        fullWidth
                        variant="contained"
                        color="secondary"
                        sx={{ mt: 3, mb: 2 }}
                    >
                        Sign In
                    </Button>
                </Box>
            </Box>

            <Typography variant="body2" color="text.secondary" align="center">
                {"Copyright © "}
                <Link color="inherit" href="https://www.clarusway.com/">
                    Clarusway
                </Link> {new Date().getFullYear()}
                {"."}
            </Typography>
        </Container>
    );
}
```

-   PrivateRouter.jsx (useSelector() ile state i kullanacagimiz componentte cagirma)

```jsx
import { useSelector } from "react-redux";
import { Navigate, Outlet } from "react-router-dom";

const PrivateRouter = () => {
    // const user = true
    //? consuming
    const user = useSelector((state) => state.auth.user);

    return user?.email ? <Outlet /> : <Navigate to="/login" />;
};

export default PrivateRouter;
```

-   Navbar.jsx

```jsx
import * as React from "react";
import AppBar from "@mui/material/AppBar";
import Box from "@mui/material/Box";
import Toolbar from "@mui/material/Toolbar";
import Typography from "@mui/material/Typography";
import Button from "@mui/material/Button";
import { useNavigate } from "react-router-dom";
import { useSelector, useDispatch } from "react-redux";
import { clearUser } from "../features/authSlice";

export default function Navbar() {
    const navigate = useNavigate();
    const dispatch = useDispatch();
    const { user } = useSelector((state) => state.auth);

    const handleLogout = () => {
        dispatch(clearUser());
    };

    return (
        <Box sx={{ flexGrow: 1 }}>
            <AppBar position="static" color="secondary">
                <Toolbar>
                    <Typography
                        variant="h6"
                        component="div"
                        sx={{ flexGrow: 1, cursor: "pointer" }}
                        onClick={() => navigate("/")}
                    >
                        Clarusway News
                    </Typography>

                    {user?.email && (
                        <Button color="inherit" onClick={handleLogout}>
                            Logout
                        </Button>
                    )}
                    {!user?.email && <Button color="inherit">Login</Button>}
                </Toolbar>
            </AppBar>
        </Box>
    );
}
```

-   News.jsx

```jsx
import Box from "@mui/material/Box";
import Card from "@mui/material/Card";
import CardActions from "@mui/material/CardActions";
import CardContent from "@mui/material/CardContent";
import Button from "@mui/material/Button";
import Typography from "@mui/material/Typography";
import { CardMedia } from "@mui/material";
import { clearNews, getNews } from "../features/newsSlice";
import { useDispatch, useSelector } from "react-redux";
import { useEffect } from "react";
import loadingGif from "../assets/loading.gif";

const News = () => {
    const dispatch = useDispatch();
    const { news, error, loading } = useSelector((state) => state.api);

    useEffect(() => {
        dispatch(getNews());

        //? news componenti DOM tree'den kaldirilinca state'deki bilgileri temizle
        return () => {
            dispatch(clearNews());
        };
    }, []);

    console.log(error);
    return (
        <>
            <h1>NEWS</h1>

            {loading && (
                <Box display="flex" alignItems="center" justifyContent="center">
                    <img src={loadingGif} />
                </Box>
            )}

            {error && (
                <Typography variant="h3" color={"error"}>
                    News can not be fetched
                </Typography>
            )}
            <Box
                xs={{ d: "flex" }}
                display="flex"
                alignItems="center"
                justifyContent="space-evenly"
                flexWrap="wrap"
            >
                {news?.map((item, index) => (
                    <Card
                        sx={{ maxWidth: 345, m: 5, maxHeight: 600 }}
                        key={index}
                    >
                        <CardMedia
                            component="img"
                            height="250"
                            image={item?.urlToImage}
                            alt="img"
                        />
                        <CardContent>
                            <Typography
                                gutterBottom
                                variant="h5"
                                component="div"
                            >
                                {item?.title}
                            </Typography>
                            <Typography variant="body2" color="text.secondary">
                                {item?.content}
                            </Typography>
                        </CardContent>
                        <CardActions>
                            <Button size="small">Share</Button>
                            <Button
                                size="small"
                                href={item?.url}
                                target="_blank"
                            >
                                Detail
                            </Button>
                        </CardActions>
                    </Card>
                ))}
            </Box>
        </>
    );
};

export default News;
```
