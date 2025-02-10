const express = require("express");
const session = require("express-session");
const passport = require("passport");
const DiscordStrategy = require("passport-discord").Strategy;
const path = require("path");

const app = express();

// 디스코드 OAuth2 설정
const CLIENT_ID = "YOUR_CLIENT_ID";
const CLIENT_SECRET = "YOUR_CLIENT_SECRET";
const CALLBACK_URL = "https://joonhaRemot.xyz/auth/callback";

passport.use(new DiscordStrategy({
    clientID: CLIENT_ID,
    clientSecret: CLIENT_SECRET,
    callbackURL: CALLBACK_URL,
    scope: ["identify", "email"]
}, (accessToken, refreshToken, profile, done) => {
    return done(null, profile);
}));

passport.serializeUser((user, done) => {
    done(null, user);
});
passport.deserializeUser((obj, done) => {
    done(null, obj);
});

app.use(session({
    secret: "supersecret",
    resave: false,
    saveUninitialized: false
}));
app.use(passport.initialize());
app.use(passport.session());

// 정적 파일 제공 (HTML, CSS)
app.use(express.static(path.join(__dirname, "public")));

// 디스코드 로그인 라우트
app.get("/auth/discord", passport.authenticate("discord"));
app.get("/auth/callback", passport.authenticate("discord", { failureRedirect: "/" }), (req, res) => {
    res.redirect("/success");
});

// 인증 성공 페이지
app.get("/success", (req, res) => {
    if (!req.isAuthenticated()) {
        return res.redirect("/");
    }
    res.sendFile(path.join(__dirname, "public", "success.html"));
});

// 로그아웃
app.get("/logout", (req, res) => {
    req.logout(() => {
        res.redirect("/");
    });
});

// 서버 실행
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`Server running on https://joonhaRemot.xyz`));

// success.html 생성 (인증 완료 페이지)
const fs = require("fs");
const successHTML = `
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>인증 완료</title>
    <style>
        body {
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            background-color: #1e1e1e;
            color: white;
            font-family: Arial, sans-serif;
        }
        .container {
            text-align: center;
            background: #2a2a2a;
            padding: 20px;
            border-radius: 10px;
        }
        .checkmark {
            font-size: 50px;
            color: #4CAF50;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="checkmark">✔</div>
        <h2>인증됨</h2>
        <p>이제 이 창을 닫아도 됩니다.</p>
    </div>
</body>
</html>
`;
fs.mkdirSync("public", { recursive: true });
fs.writeFileSync("public/success.html", successHTML);
