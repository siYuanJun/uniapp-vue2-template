<style lang="less">
    .cu-load {
        &:after {
            content: '' !important;
        }
    }

    .text-footer {
        position: absolute;
        bottom: 50upx;
        left: 0px;
        width: 100%;
    }

    .login-body {
        padding-top: 45%;
    }
</style>

<template>
    <view class="">
        <view class="login-body">
            <view class="text-center">
                <image :src="wxlogo" mode="aspectFit" style="width: 220upx; height: 220upx;" />
            </view>
            <view class=" text-center">
                <view class="text-black-000 text-xxl margin-top-xl">微信授权页</view>
                <view class="text-black text-df margin-top-sm">授权并同意使用微信手机号登录小程序</view>
                <button class="cu-btn lg bg-theme text-white margin-top"
                    v-if="!submitLoading">
                    <view class="flex justify-center align-center">
                        <view class="name">登录中</view>
                        <view class="cu-load loading"></view>
                    </view>
                </button>
                <button class="cu-btn lg bg-theme text-white margin-top" v-if="canIUse && !loginToken && submitLoading"
                    @tap="getLogin">微信授权</button>
                <button class="cu-btn lg bg-theme text-white margin-top"
                    v-else-if="canIUse && loginToken && submitLoading" open-type="getPhoneNumber"
                    @getphonenumber="getLogin">
                    微信手机号登录
                </button>
            </view>
        </view>
        <view class="text-footer text-sm text-black" @tap="href('/pages/article/content?id=1')">
            <view class="padding flex justify-center">
                <view class="">
                    点击登录即表示已阅读并同意
                </view>
                <view class="">
                    《法律条款与隐私政策》
                </view>
            </view>
        </view>
    </view>
</template>

<script>
    import {wxLogin as wxLoginApi, userInfo as userInfoApi} from '@/api/auth.js'

    export default {
        data() {
            return {
                loginCode: '',
                canIUse: '',
                wxlogo: 'data:image/jpeg;base64,iVBORw0KGgoAAAANSUhEUgAAAIQAAACECAYAAABRRIOnAAAABGdBTUEAALGPC/xhBQAAACBjSFJNAAB6JgAAgIQAAPoAAACA6AAAdTAAAOpgAAA6mAAAF3CculE8AAAABmJLR0QA/wD/AP+gvaeTAABLS0lEQVR42u29+a9l2XXf91l773POne8bqqq7quduskmKzZmUyJAaLNOWY0VKTEiBY0OAY9jxL/4hCJK/IECA/GQE+SEIHCtIYtmRYytybMiObMuSKJmmKA4im+xmj9VVPdTw6g333eEMe++VH85Q9w1VXd1VPVDIIh7r9X3v3XvO2Wuv4bu+a21RVeVPiagqIkJ7S+33qkqMkRACIQRijN1rMcbu70XkyPfWWowxGGOOfH/8d9c/+0dd3Ht9AfdavPdUVXXk31YB3kxOW+TjC90qRZqmJElCkiRYa/9UKAOA/KhZiOML1C58WZaUZdlZABHpfm99Ydf/tn2t3fXt7x23NKxZm9Ouo1WQNE3JsqxTkB9Fq/EjpRDtpYYQyPOc1Wp1RAHWf2fd1DvncM5hjOn+bZXgdp/TupR1d3Mrq7P++a1iZFlGkiQ/UkrxvleIdpd57ymKgjzPqarq1N2aJAlZlpGmabfw6wt13Dp0D+GYMh1//bRrijGesE6nxTDOObIso9/v49z730O/LxVi/cFWVcVqtSLP825XtqbfGNPtxDRNb7vr3w0py5I8zymKolNaa213zWmaMhgMyLLsfWs13rcKUVUV8/m823nt68YY+v0+vV6vM8fvl4e7/iirquqUI4RwwmoMh0N6vd775tpbeV8phKpSFAXL5fKEIrRm9/hDvJVLeK/vo72WGCNlWbJarSiK4kiamyQJo9HofaUY7wuFOG4RaB5k6xKGw+GPXHB22j1671kul537a8U5x2QyIUmS99ztvecKEWNkPp+zXC6PmNVWEZxz7yu38HZl/TG3irFarY5YkyzLGI/H7ymu8Z4oRPuReZ4zn88JIXSv93o9RqMRSZK8Jw/kXt5fCAHvfRc3rO9+ESGEwOHh4RFXIiKMRiMGg8F7Yi3edYVQVUIIzOdzVqtVl0045xiPx+/rCPxOpSxL5vM5RVEcAaiyLOuUfT2T8t4zm806d9liGdPp9F3fGO+qQqgqeZ4zm82OgEntjqABlH6UpSgK9vf3CSGcGvBaa9nY2CBN0yN/F2Psns06uDYajej3++/ac3nXFOJ4rNDugo2NjR8JwOZO73F3d5eqqm75O23GtLW1dQLebp/LbDZjtVp1r/X7fSaTCdbad/we3pWVCCGwv79/xCROJpP3zE++U1KWZecmTnN77eIXRUFZlifcY/t30+mUNE05ODjAGEOe54QQ3hUX8o6vRlVV7O7udspgrWV7e5vRaPQjHyscl6Iobqvg7f2KSPc8bvV7/X6fM2fOdApQliW7u7tdZvJOyTuqEHmes7e3R1VVxBhJ05StrS2yLDvygP40yZ0CZG+2qG1tZmtri16vhzGGGCMHBwdHUvR7rRzvmEKsVisODg664Go0GrG5ufmnJl44TVoffyeL1NY47kQxptMpw+EQmjhlf3+f+Xx+BNy6V3JPV6fdHYvFgvl83t3saDRiNBrd84u/l9fdyt1YrbbAdquFaoPIlj9xnGNxKzHGMB6Pcc5xcHAA0D3fe+167/l2XSwWXTAEsLGx8b7C6k+TGCPL5ZKiKFBVkiTp4PK3IkmS0Ov1ugzhuLQWIcsynHNvuf7Spp/7+/uoKsvlEpoNd6+C83uadi4WC2azWfffm5ub73ugqU0VvfewZi2stWxubr4lpWh5ErPZjDzPT/xcROj1ekwmk7e1gO21lWXZKYWqMhwOGY/H9+Q537VCtFq+XC47wMkYw3Q6fdcsw7ovXo/k7+S6Z7MZi8Xi5INpkMWNjY23FCS2bqCt2rawvLW2q9a2buNupCgKdnd3ERFijIzH4yNu+e0+97t2GW1e3fo2EWEymdDv99/R9KiV1nQuFgtijDjnGAwG9Pv9N30w7cLd6mdlWRJjvCNAaB2Gbskxt1Kme2He0zRle3ubvb09RIT5fI61lsFgcFfP/a4Uoq1L7O/vd6+1ysBdBWgKCiAgTYAWDWpA2nsVQJX5bM58uUQJgBIrZbF/QCxLRpNJ84u1BLmZVmnzE+HNuZWdxAgCqhCNYKOgAj5G8sUhfrWi1ECBkmCJMXbZQSv3inzbMrA2NjY6S9HGbm1az9vgibxtVW395d7eXleXaHH3u71ZJeLFEwGPx1Oh2qxibL5UCWXJar5CaTRFFB+FKjoO8gV59HgiHiVI/W713ytRPIqS2FvHCMcrlEWjE+ojBwR88BzM5+zu7HG4zFmgeMAhoHUm0FY719HLe+lGsyxjOp12C39wcID3/tQWgndMIdqdM5vNug9vy9bGmLt2FdoohQJGLaIQrdZ73YAaAGFVVhQ2oFLhYiRqwqua8CwJu3HIrLJ4NYgKJhpMbK5dFEeJSMVg0j8Vam4VfF0Sb8iNIFXO6Nnn2Pvdf0V45tv0ygOiCBUDEj9iWPRBb9Zv7gRvuBvp9/uMx+OOIX5wcPC2P+9tuYzWZ7XpVZIkTCaTe2YOjQJqKaX+19J8L/UuNQojBUWIVjHBIxHKJOMbr9/g6cMSJzC0Vzkz7LOZGbZHfbYGjmGWsZ06Hkh6JArO1FD6unK3nMcTtQZTkj/zDJd/9f8g/JuvQl5Q9FPGP/0Fxl/6Sfqf+iTLwYAitbgAgnScyneyMCUiDIdDvPesVivKsuTw8JDJZPLuKERVVRweHtZv4NyR4OlemMMohhfmC3792dcoJaMfSyqj9KqUPCnZpuJXPv5h+kmCXYCNQrTC3BguLz07bowai0G5tAjYQ4/dOUSpUFE+aDx/+7MfYtMaxNQKsLW11fV4tKnm0XtRVtef4fJ//d9gL77MqIr4pz7A2V/+88xfvMxr/+1/h3n0Uc7/7b+BfPQjBHr1XzX0wHcaoW2D+bZvZLVadQH2OxZDtHFDm1EYYxiORkduVte+bvEub35zKpSV8lwufN/3eWY15IXlkO9WI77j+zxfCjmQpJZpOiAhpcJwLV9x4ANWErIYSGIAEYJNKGxCyYil7aP9lCQVos3xUuMPMUZmhzN29/a69PmI2VVl/3/9TYbPvUKiyuFPfoyzv/JXeP7vf40s2eD+r3yB7MVnOfz7/wxC+wyksRL5u5Jxtel+a6nbzOstvcdb+eXWVVRV1YEsSb9PRBAVAm0QB75be0VRUCVSUT+tWIfqql2MWDvdADEgKEHAKgzKQOUiUUBNRRYsgsM2WcNoMqE36TOQPpfmnqvDAT0fgNh8bgQCohEXhV7wPDztkwGiKYrFh8jB7oxVtaCUguWy6ILldZlfvciq5xh+/qd44m/+dW584wece+Eih//bb5F9/NMMvvLTLF74E5KXXsIAK0nJKkP0RR0RvQvMkzRNu9inZWK9FWV8SwpRVRWLxaJrkhmPx6SU2BggKlY9iQYSD4nXJiQPgEelwoQEVUclhlIEL4LgMZSAJxolGIhyU0msKtpkEELAqoLWO0/bUvFwxPDcBtcXOUHbn7RpZZOAoBSZJ0rJ2WEPkUg0isbIau+AuPKkHvpecVGpyoqDg4Oj3eH9IcOf/Tzm/jM881/9HdzhLu4nPkLyM5/k0m/9NqvvXWb02Y/y4t/7nxke3CAVTzARb94dZaCx4uvxT57np6Kmd60QLZOHxry28KsiLI1h1wn70uNQEg4tzJxwwxoOxaE4RG8GVUpsFjciIdbGAUMZhNWqYDmbU1QBkdq+3Pa6mp9fKTxXVp4sWLycbiZ73nNfDk+Ox1hvkGiY5xU7GGaDAYVNiALBBMTUnIW2XgDQ/+IXCJMtih9cJLc5u7/7VQZf/gz9Dz7C5r/9JsWrNzi4eIXJRFldfI4krqhcpLLSXuwdPee7kTZjWq9vHB4edojpm8mbRjrr0HTbPNO2owHkkvIPvv08zywFEwNJ9HixdRpmI08ax1/7zAcYiiW62nxblIhjhaEyKYtK2VvlrGZz7k8M/RBYBdvd4O0fgBIxvDibc6CWJDiCLW8CWGuSm0Ce9fj6tSXPsiT3noNVxW5ekGZ9Pn/fhPM2ItJYuKaMPxgMMCKk0zPsvHbA43/rK4Q/+CrmN36Xxdef5vD6AVsPnGf7K38WSXrc+Lv/iPwnD8hE0GhwpnnMp9zKabD7vcjU0jRlOBx2yjCfz5lOp3evEC1W3pIyrLUnSq6zkHCJFCcWZ0uCCKKWla3YFKW0QiYewWLVUYrwL75/mW8uPAceQukxEnjAen7hww9jQwXcWZqmzf9e3J+xshlpZcit4vSUByoJ+yT81uUDolthY0oaLHkywBWRLZ3zwIUxqjdNbAtFZ2nK6PEnePxv/GW+8d//j3z2P/9L7F6fUf3br1Pcv0X/b/0Kr/7dXyf7j7+E/aVfYvDYI+QkZD6h77I6djpmkFukt+WM3MselHbj5nneWbp+v0+aprdVuDvKhRaLRYe4tc0zrSQRgjEYsZjoqUxANEFUMcFSJQUWRSVicE3wCK+jPFs5VBypDUQJDJMKwYHom7qKbo1VOYjwwr4nkqGmQhQUgxAxqg2aZXEERCLBWJQBqKMSxYtSZBV7RIyPqKuApNvSIQQU5eBb3yL/2rN84Of+DHu//zWuX7rMxiMPcPbMlOd+/Z/y2Bc/RzxzhllxAMNtbDAYF+glFqPmiIWIMXJ4eNh1cbVx2WQyuScV4nXX0VZGDw8P2drauu3fvWkMEUJguVx2sxVauvzND4aqQY6tepBQ37cEXKxvyikYlbr8QMQCWQZWI2mIZCE2lkhwVcToW0iV1HBlWXKjqJ93NFXz3JveSqlVK4oh80LmIz0PvcpgNWLVMCgtqYcYDL3QI/H2pH1XZfXS8yy+9k2mH3iAnd//OmE249DB4c5Vxi9e4srrr1G9cYPq3/4BkgxAwfYdzhpEjirEYrFguVx2Qet6Xeh2rO23Ku2MCpqkoCzL2yrbm1qIddr8eDx+25W69hKiCBJhq9fHMidIRJusImidKupb2BzBGF7enVGJWXvegtEmWxGhxqwrVsa0VbEaJ5A69U1DnZnMypy5s4jLSBp+BE1+r4CkGcQFO3/0NTbTlMEnnmK2XJDkc5IKVsDilVfZcAlJKuSpYzDZAI3dPUljHVar1S3ZVXmen+jbuBsZj8fcuHGjsxLrxa87UojWx7TWoR3A1VYx366o1thBKspmliG6S7SG2FUNQ123WruO04NupfKR6IRChOd255RiEVVE2xS1dhUqAUvENgiJ09oiTU3CyFpIDIMUegPDhV6G00OIFSI3Y5jWt/fOPMj+wHH2ldfIb8zY/70/JIlKIADC8IMf4I3dXarXX8H+0b9n8y/8IhhTF2/XFGJ98NlpNLp7aSHWB6kURdENXrmVUrhbvQmNdWgvvK1V3MWlQQNYpQoTC1a0gY/q5fMxNlai8YFtrfmYxBjZ3VtAVjKzQ15bFEQ7xkZdcxeKSp1apgSGwfPFzT6PbW8ycsKFjT59a1hWAb+/IlDhTclGXlFEpXT15wwGg6b5FjY+8xM8+1u/yYXVEofQi5G+93gDpUT0+nXcYsZoseTK3/tVtn7iP0DHY1Slznqax7deADzO57xT/sVbldFo1PWQLhaLI5zOI0PVTv1rhRgiy9UKjGATR5alp+LS7bdGFcGgIkAdB9iG03ATJqo/UIFpkpGIYBWCqQtWXoXRuM90MmE0SG8iS2ufJkTAEgnEyvPCziH70TQwsXafpRhQbVJIz0e2hvzEfUMeNTnnbck0URJXMcgCPeexUhKssp9a5mla2xdjulqAIrhHHuHDf/Wv8HpZsrSgUrKwOct+REYODmf09ueMi8D45VdYXXwB1QiNcre3cpyzcHwz9nq9W6+snvxvVaia0rve4j1bvicNBa+lDB7f5LeMIYrC46MSRJkOMoxRVKsji6tAJRajgSRG0HphkDrSd9E1SygYNQSj9FSpjLKRJCTi6JeORRboeUNOiiaGYTog8cvmk9bmQqE49aBZXeVU4YWVIbd9XKyBlyh1RC9qMWZFhSXTwGe3HMOwBE1I0jFGHKIRK0q6Oaba9/R9/UilmfgyXevBFMCIcO6n/xyD6TbX/uE/Yvns9yh2rtMrIlIqdmcXt5gzywwHG5s8cO4MofIsS0/0S6KAtfUYw/F4fGRuZistyngnugAtYajmdxh1gBClzrGOK8VgMOgafdri1x0rxHK5wDTJ26g3gChI55sbtURJ1WDxeCtoyz1QobCCE0VNWbOepMRiahshSi9NwEIQaUseSAdZny6CIUrLkhEWJuH12QxtUdDu5jxqayxh4OEjW1ucdyn90lG4QDoEFY8JFsEgVtna3OrSQNvA8sd3qkhErWPyic8yefLjFKtD/OGC/JmX2P32H7Pz9J8Qr1yi108595/9VeL0DLP9ObZSguSNpaiNcq/XY2trq/PrNGDS7VLOQB2LSGfatXmcSuab4o4FdbH5jaPv087WbOd2DYfDI4PZTihE608qX1GFAlTpZ32suNrHY+sLalawrRpEFG9CUzMAFywKuFAXqCKCJ6kjbCyhCewmRlgeUXuhaoOMU3eDJRiF6BEVrvjInh4l1qpqg5KmDArPh/uWn74wZpDv8DITtgYjzojDhEiU+n5sEzRPp9MuZjj+oGqxGGMJiRKTFJkMSe+D7InH2P6LPwXlCr+YoShLb5iRoEFRItHWG6XeSzX3sp2Xtb6LT2M6tXGG09j57DZaCmIoxHAo8MZqzsRZHh9kYE4qVWslZrMZIQTKsjyh9CcshKqyypfEWCFq6A36VFIHgxklxOZijCUqzMWyTAboypNE8KbW4swrimUWHH9yY0GISgyBvDI8NHU8NcrY6jmuz48aRV/FW6OUaoimjlc0Cq8sCmZiu0CoNb0GGBeej45SfvnH7ue+VHhxd8zff+YyD45G/PwHLvDwKENFsaJYbirU7dK92FRgTaPaEupabeXAmARrMkwy5cbuAbn1uFhjLWq0pXkdWfj1/z7te44FnbGJjUqBhcLVec7lvTkv3ZjzcrHidYWPZpb/8uNPMrgFOrDeRrlarU5kjkcUor3QujqmOGeRJGEOXDucs+OFw7ziMC9ZBlh5g889l4sKqz2SUFKZSGXrTKE0wgzDrz19mcImTfA/5JObJU8+9SC2V8cVNKYQlYY4dyuUsnFYGhExvLF/SCUjes3OaRwSm4nwsw+N+PwDZzkjntfLyP/+4orX7RZXV4GXvneJn3/iLD9zblJzJuyd9V6o1PDzQuHfv3qFjdGER8dDJo1lVInMlvuUkpOElDQGokRKA0Ydwp0UmNZZxLUbDA1N4PXc8/Lekhf2FrwwW7ETDSuxGE0wvscqsby8OODpg0M+d2ay/i6dcllruxS0qqoTbK4jCtFNbK0CGMM4SbnsI7/+/Td4ZVaQN8COIqjUYI1TxYtF8FRGsLHONLwFqxWugtz1CGKwGihs5OW5sBci94nhGzZgo5Bby7iMHAZPUHvqg4p2iY0O0YBKwlObA7Zy2I+ClciGFR6fTvjw/RuME0+iyo43/Pr3L/Fq1cMbxUbHVaP8g4uvEb3nz144QxICwVQIGdYLPom4UxIwVUF94OlVyT9+aUGkIBtc41ObE37ywoSHU0NRLBu01tZxChEXLRGDwddxiAqJpIgKKoqP4SZQ1lTltHLse89LhyueOVzx4t6SG7lSBCUaiychmjaIB7WBLAZWJuOfvbrgE1tjMIIDrAds624MvV6vS0Fbt9FaphMWIs9zEEOUSNLr8cKNfV7eX7JwfUROFux8678bhFGQjn9QZxgWbwxRmqAQWOWRazcOuS8ZsJHPURM5yAJiPD5WGHO62VZMnUFQYkLg8c0xHy9r4Me7COrZGqYMUagsc4R/+twVvpOHuqQtERstCZaVyfjt51/nwnDAU5PezXxaG+zjlLjOqVAZyx9feoOF60G0HPqc33/jBj98/TqfOzPlE5sjploQxVKKqzsJtH4SlUnqpyKBQd+CKZEANjoqY9gPyquzFRd3l/zJas61+ZJlNJQ2oxKHE7D2pvU0umZN5WZK/+psxXduzPj09hSrAbUgjWNU6gEkLZUhz/Ouh0ZETsYQdeOKolaQLOMg3yda2wQpd0b0kDr4pRKDF4NRJYmRykWEnNJGfri/Qxxts0oEISF3ilVhVXpoLIQcWRWpX1dBVEmJDGJJgsfXxRKiGOiNiAKVGv7J63v87m7AMwJbdpmRDQZjMq6nlt9++SqPf/xRBrhutwlyy/u6uCp4bn9FbjMShDRaSiu8JCk7OwVvHKz48qNn2WLVFNWkLrJJybBKiGqJSQ/Xm3AQ4LVFwbMHM17Zm3P5sGBXEwocSg9DDxMjaVQyrYtwegeVg1IMv/XKHk9tTEnEUwo4tVgFMdJN72kn74YQOrDsiEK0w72NCFmagRh8FKKaU/kFtxU1iEAvlpxP4D6rnCFl3LekgxEPJTlvBI+1BaUREo1kPlAuA7s3FhyUdd/E+tq0VggrjCcZAwIaHCZNGJiEzKak4pAIl5dz/vDKq5R2i0kOS1snq95oA3FbCmN4YX7ITl7w6CCtmU3mJuPqxGYR4Wtv7DMPQ8QqwXiGJRixFGnCXmL5drWgurLHVy70GIQcJ5CLsMj6rFLhoIxcLUou//BVXp17dguhlJrKF02PaGprklVN45GJeAl16t7gDG/+6C0vrgLfu37Ij98/QPBNwbF5jk3w3DY3e+875PKIQqxPjx2l/Ua7a2zBqhLvELlWMQQMVj0Tv+DPP/kIHzAlWQUZc5YEhkHpAT85tfSdxRvBVsIZF8n9jBhvspabd8UQm0WNmJ6jn0xItalMNnUPlfp3HpgM+S8+8AS/+cMrvJKUWG9wYimtogSSKKQBSgwHQevsqas3nH6jO5Xn29dWqI7IwpLcVRQ2QQUGPq8Dadfj+3PPUwvDh0d9errilaXyrWs5Ly8LFpVniaFyKYVJMVboB4MguNBC3ErpClSEKNKFyy7WRbs3XwDwJuVfX7rGh+5/gs0oYDx1PnVzJuZsNsNaS1EUXXZ1RCFazoNB6Vlbk1iNZVRV5CpUNpAYQ2qFzAiJFUZUXKoC89jHRakXqzGTwShRE7Z9zsCsKK2QawVSUBDpi+UXP3gBU+TMlznRWERLjAZSrTAaqYzUoAtgVcmdYJsdbNA6pWt8Y+yotUIW4bOjAfd/9lH+r0tX+MEbMxYyJvOGIJ4qgawQ+iEym3v2liuKpMSYlIGxuCzFZT2cgKgniOHfXV2wFwyGiI2GJCQEsXW81UD1IRpKcXx755BHNzcQD6vS8d3dJTM3RIioVYxGBmXAmDpVB6lhf7UNxmMwsfb80mxMGn5pMA3PlLqd0MYasFOp3xeBQhJeLJd89/qMn9yeYCTU1MWmaNceGdEGlq2csBAighiLNJr/6fsmPDQdIpljapRUhESEtEEiS2P5H56+yAsHNfASRTERrAZKoxSmh1NDEIuNAgywCiqWzBimTtjZX+CoAad6cVNKDMFYVtbS9xCMUuAonBBDRUBwawilrHVpKtSKkgQewPDXHr/At0Yjfuv56+xHQ3ApMQqVUwZOGbmIVAEVz8oq5Ja4ysHmjFKhN8zY1ZTvvHaAjUqUZV3GR1CpFSGKaSxojYVcryJqLaO0zydGm/y7xWWeriokGNIQEWomeVizR6La8tWP4BYqtVWLpo6fbGx6UhuUMErAm7rUb6LFRBj6JSPj2bmxT9ie1B1w5ije4Zw7Mpz9hMtoy64mTYnGkgAXevBATyAUqBs2QXjNjUQUo/WCucipcYYg9Ps9xBdHCpci0o3xzbKsG2JKQ2ZRLMPKE1li1WOirZlZ3pF4T9UAZKcZd6EBNgyggQ1v+MnhgK0nzvLbr+3wg1WFxhSk4JFzfTZNgVBbJKO2BoBUMX5FXhXshhHfmadcX3m8dTX4ZmNTs7FdENxKFGGpKb10wLQPxht+8QNneeN7bzCPGaKmtgANaHWn/I/KKC4KSTAYFbzRRhHAaCT1nl5UtnsJP7Hd48kL53mgl2ClduPHpY0j2pqKMeamQrTKoKpkaYIh1sVGSWpjbNO6JAEgpv5SSFVIYltwXluQVrtVGQ4zNmSDIq+PQHLOdccbrHP/WrHR86AoP7c9Yu4iiEU1bRY6Yah9pJxT9QY4HOY4ZqB1DBYseFHy6pByvuS8KP/phfv5+n7FH964ylQrvjA5Ty+sCI0JNrHekdKQbKTp7r74+jUqZ6icwUeHVYPzFUI4NfKXpju8bgOIPGKUn7t/i9+5fJVXBz0qSZjmkAYo7rCpK4mKqFAZQxSPtyVWLIOVZUs8H9pM+MQDmzy8MWTbGawqRmJbmjuxLusTddvusu5S1suhibM3y8wiNU9awFEeQxEFJSEI+LVStRyzBChkvYxe1j9BClk/WKSjvIsyNZHPbY3waUWQChv6iOTYkIKkaLFAdFDbiFPNhGLUsFqtOJwvMBZsiGzGJZ/fzjizMaWfVzyIgloKSxM814ikquAlwWmkFzw//cT9nJsv+MEb+1zzKSszIOKa0rg/8tEGZRALxoklEFjlc4rDAz41GpA9vMVvv3GDq6KosW38f4cKUeM+3kZSLTnjVzw26vHRBzf40NkR51JHTwMGj5JiYp2Ol6Ik2izQ2se1NZv2dKDOZbSpR7tA1rbpTR3ZmyZk06ZARQeoCUHqHeVN3WNpWxi32WGGUPvWCEECRgSvwkoEDYFxg3aNhmN8UZHHCtQSXIL1io01Ld7FgNqA0wqioz/awMW0LnAe81XaXlNe4feX9HHEGKhsyn7qgZKPWCEOLIUBQwbqMYCLdapHUziKOFzIOceM+waGTz9xjudXyjdvLHh5lVOZFFXbBISKJWJixf1DYWCFsvLcWC3IrKOncz7W73Ph4Yf5xvU538sP2TWKqIOGc9GumFMojMEpxMZa1xuzvtftEPhrH/sAHxw7MlthCbWHFEOQpMYkxdTPoqtOy5HUsz17rIUbaIPKdYWAmvVVF1sdiu1YS6YtTksdEUcUVaEXYseT6FV1NqIINoIj51VVykXJ61XkYH/O3kz5Xsx5wHj+5sceYx4U9ZY4mvJ6vsCthKpp3Y/lTW5A7vtEVfr9hCyA35sRotTX2KbNKAFhIZGDwzkhNIsbIrlxzaOFGAIhRmDJF8ZDPtiH0gpWHQGIJiAtHd8kmIbmt6lLPtc3fOap+/jBfMW/v7THxVnJyqVUph41OEB58uwUFIrZAm+UpHRgDMYUnHPKly8kfJZtLhclr62Uw2XBqgoEMTgx7BfworOkRSC4CokJpXX0wxxkyPnRgB/bTMlUQG6SgmU9UzB18JlxOhWqpUa2MUSnEC1/sm2FFyzEGnZVqTkLEUvaaJk0UZBow7hOBsiqLuSsBzmoZWF6/Op3LxP9nDycqSuG0TLPMrZTZWksv/a953llbkCEpYn0V7VS0dRXWqgod9oEtTf5aJ4+gZOEEokGod8hj7Wr8l3ZHqmnwPV0yYd6SjU0qKmHiggRF9rHqkDAEAlYgghGPf35nJ+ZbvHZj2/ww4MFf3DxGj+cl8xMH5skfPTcJiZUhHKFcdJYAFejuAoZng1j2Bw4PpNB3OwRxDKaDEl7lt94eZ9Lb+xj1VGpA3VdyRvAGotoRPXuztY4PluzU6b14VixMblWFRO0dhmi+Ka4JU1/pUa4UkWuzAssCaFLfwzBVA2dPWNWQWlTxiHHO48Jjn5Zcbbv6CnsBeFlm+A0waMMnSPY5uQ9K52lc6Ge/tI9AAUxHivLIzepCMGtje/psMd1xK5+j1KhtBlBAiK+4djU5XU1lmgiIh5DhfEWtZZSIsEvGPkhU5fy2c0BT24+yvOzJd94+Q02p2MupAZdrVCURrdRdfWVSMRqpKcejbEhBcMg7THpDYmi9LQkJTSKYBpw6ihHQkVuCaK9maxP3G//O4SAO354qTQMpkDENj2SAUPAEr0wj5FXVzkvzua8tlpycWfBoR8goiChRsPUEsUTTcSFSOotaXCIREqUXkxwXjl35gxWIdGAQ+j7iDd1jm7X6elNiOBPKYK2dLkjr1Ez7088BEmP7LL6QVSo1pXYelCJYZ44fuON1+mbHk9Mt/hAf8x2WBGbHtSokWAiB7pk06WYCBsm8Kmp42OfeAhVoa9QJQnRJiSRrtSnUgNo9ewsRaJgcbgkYTzcwKohap2ZBBQrNXPc4I8t/t13D7cuo1WIGOPNGKJVCGstRgWDRcWyFz0vLwpePDjk+YOKnVnB3CulOKJJqZzBWMHEpoijhsQbgq17OytbkgRwUYia0o+ex4zymSfO8vnzKaKe0ri6syqAt5HS0YBYx8zbafy6lqbVZi1NoBuOMYbqkO1kTC8h4DSSxKZ3Qhy7S+G5xQalCt85mHM2DXxwu8enpynbKmTeYSQSFx7tUxcCG6DMarOTVdDEMt7cppwvKfJlS4Jr4OgmFRQh7Q3q9khbK1ttYWtX7U1dIhepald+Qhn0tvWNW/20tZ7rx0jGGHHtN61iGFMXsoIoEc8zV2f86su7qGYsrKImwSQtnKrYIJhYw6mVcVgM1lRYBR8dVRq4P5RcmAgfHmc8fuZ+Hhn2GDrFkOMVPClJEMokx8QM7wKqjp6uUMkBIQ0JqW1O5zUWI4IxgpUSsRVTC0kCwzDkq4dVPYbg2CY6rWilEvE2R8Ui0bB0gednlkprl7ECXo3Cq9cK/ng/8OXRkM+PE6xdkHqHVAopNcurje/0Zld65hyD6YTVsE9ZLAlVVdeHxJAmGWmWYJOmxtAEiAIkvgagSmvoVwY1ShorAlmNEYlSAWlcEczJGMqi3VgF1YjFYMQcmbp32tHXJyARaUkPEnEEnrh/i+TijANx9EPZVAxAm0zDi6lrEChJqCuJZcOINiqMCs/PP3KBj24m9PuWoTFkDWciSspyleNDpGaZxdrSSMTpIV86v82jg016ImTBMrYLNsYD+llWK4Q08LWpQZjKwmqpPPeti1yR09Ce0/ZK7dObhJHcGi7uzxu0tE5Fa9PhmBXCby2vc3ll+DMPnGGbkomJJMcpf02M0np/iAycIUsmaAdLtg6gtlx1Fm87O+alttTaKHJEuk60JAgmOGK0EHsn4gjp+LEttdLWHuqUDsUuTGi8xKkYWZSaFitB2BDh41s9fn+vpDLrfN+Of10Xa4Ih9TWos0oixDqHToJnkBdkB3PKRcT1BmSjMVhDkQdms/mJZYooTjwPSuApauzBi6OySihzesM+1jTImwYCASFQibJKEpZZCaXlTkrFtXmvmUfeGG5UytXcQ3KSpGNEmWcjvlF6Di/d4D95ZJv74q19udHafUUxuKgY6g0VohJVEDGIqYeo1LtZqIA8wkzqCufNKrOpy/ZSZ3FXq8hvv3SNTAILc9pn13/XFE9JEL50fov7+ketyQmFWA8qW6axUkOkqCOx8PnzU/545yKzZIrESKqRJHpsCExczfhdmBQJtutZc02toXTCMvHkWU4ShNVyRuIM/cGQxXLRafOR0SBqIPZqgEo83kJuDFkQXHQ4tR2DOTYWzURhgKKVpV84ZneaiqnU0LcIlRheO1xwIL22+/PIrwZRhoWjsCnPcsg3XtvjyenG6e8ba4zmdV9xzQdcgIMCZqvAPM/JyxIflUqEAqhCYFWVlBGqqMwrxegQ0XoCjVFTg34a8Ua4FCOvXs3rCTsn9rU2VqWG3k1QUhU+egbuO9aN2bqMUwkyR3eNq3mTUvHkpMcjKVwq5/Q1ciFzPLEx5LHtMdvjAf/wmUv8SZFTmT6uDfyaQR4Gi6ipJ8VhUCMcrhaY1OKrFeaUokvd21F3bgejWIUsGFRW9MYZanxTbawbc9qULEqkTKBIbhYN78A+dEocrePSjQNW7jwJqxP2xShE4wFLCJZvz5b8RFny1IlOqzpP8+L4l8++xNfmFbYcoJrgjRDE1P0oxqISEBOICqpZ40qUzJSYYDAaiM1IIi+2xiubdscgjojBBdPVktp/o2lnbNUBt4lyRwODOmCK9dy0hVCbOv3AwK986kneOFhwZtjj3CBjKM1sCCCJASEFiXhba3O7w4yCC4bUu9qKNwViZx29rMdhUQ8HEW1gVgEhUrpAZWtXkPl6PkORbfDszooL54act03nlfE41Q5W91JjkfYOJwq4aHCqCDk7mvJKUXd8BRsbEEkafkeNv5e2ftCZGvZdj2/uzPjQpEfS8DBjA9kTBG/gqhfmMoYsI4lFXQxruA8qgglCElxH5WzXIdhaWYxwswfDVLgopEFQ8fUmgI7lvqbbzU8U0YixUDYzOk4YyGM9pu74fMkWyGmVyVEjlo9nhsfPHRtJ08yFCDbF5Bk25pTOY2Mf0ZrUikSCgcqYGuDS2rwbcYzHG1Qyw8YVLho8sR42FMFGgwu2XmwJlD3LtxeBf/7i6wwv3+Ant7f42EObTPuOzVhhoEn5As6n5E7vKITIPM2gs8ilueeaGZFYX7OvqLvVdK0/1TRNSi3Qe/VgTsk5XOOs23qEGKVAWSoMKkswVe3e6sHHdd9r0+PRtjy3NZm6Hmi6moqJNX3WxYgLfYIm5EmOiuKC6zCbts4E4I0DjThVkibrOK3Ovq4QR8rf74a07f1tCxnAdDLB2oN6RoQYENtYFUiC4FQYjCZUvQG/8/zrXBlsgFj2riz4N9dnPHlmg58/P+SRYc2kkoZFlN0hbiMEcptSypC9nTlGEkoRXLMIKpFoQrcwxyP6+Spvhi1qs4dCXecxgcIrRYgEcXgRooQTFLi6LyUeIb3oGtFmXaw6giQEY3AxggSCiUTxdQNRvNniVBOGlKzJ6nrW4m5BjF0HJd1621iLSdyLoVfHP7BRe4bD4ZEpNF4s3g3wYglERFOCKVlkOW7qGG9Pic7wjRde56AsSZOUiLIyllIN37yy4Ex5wIWnHsM3C5OwullBbCyeGEHjSauRxhVJdEDKudTxYFmyU1XM3bA2gUGwJA07quq2YAfkmHpmJhHEtKOQBGKK04iEOhYrbV01lmOAW829iA2/RIjGEJr+9uMSjKeSFf0Q+cTY8mDfkVghNSm9JKWfpPScIzFCZpXUQGosiakLZuPk5Iyr9fXuyt/H8ex7LYN+wsZ0gLWG5Fg6Z1FcDGTek8iyZhtLgZoFU9kAY3hl4fn9KysqcQyq2rzGtrFEI08+dF/NAZCUsYWvPPUgeaXM58VNhQTEmKbnYr2VbsADPaEXPR87P+IRk7K3WPGD2YI39hfse8PK9KkkbZqJ679r50JtjgZkWjuCuuZmO3/eU+HDw4Szy5yQ1OpqTlWIOs5SHIW1zDXyaskJmN1Ei1oh1ZwvXXiAT28PUKlTW2rCeO11FKyJDQbSNEdrAHFdm+Q6OtlKpxDruej6EcxvWdYIMp0/E+ilhl4vaz7jaP+ii4GnppYnhpY0VTbcgHFm6SeWR4nM9hf8zuU9LpkBUYQsBoxqzVWQyON9wxPjlNRXREnIgE9t9BGJHAw8+8ucPCqLPGdgYKvvsEGpqyeKiqth55gz9jkTHI9Z4SNnU3bPDnipiDwzK7g8P2RZ1TWd0hjUGGyMPLQxxkU9gswEBExg4OCXPvEoFMpytkuhoQOk2owgNrGPiRZNEtyox/eu3+DXLi4Jx1v1Q4bDIpozTQyWgKji4lpltrX2dXdOG53UTc2nlMLWj6Q2xtRBZYtpt+drvhWX0WlnM9NJm4ZYmuBIjlnp4+9rjeOXP3DuyE72hedw/5AVc14q+nxzXmA1JVNL6SImWhKfQLLgsw9MGUrdLBMksqzgxiLn4nzOq7Ocl2Y5O8GwiPDZccIvDi2TMqe0m1itSEJBZaBwSiRDgiWYEhNhWyo2eoaP9xLybcOLh5YXZhXPV56ZRrZC4CMbA6Sp8Bo8tuGRGLGIhSkgmTDanDKfzzuqYIf/OEiCIeslpBNLJoanbxH/5EmFjZFoI4kxVMaSBhB763TS3K7O0RiAdl2stTeLW9ZavPfviMt4M1lnS+d5zuHBAS4ElsmYr17fQ32KdfXwkp5XCgfLzPNIDHxqe4PdZeDfXZnxw0XJtVXOYbFinlggIWqKaoJgeX0WmJ8fMpUSpysqK4gpyYKy0BF/UgRMueRzw4zYTMIzTaQ+MoZPbid86bGzzBRe3j0gPzjgwjChEkiUupGmxrq7vdgCb845ptMpo9Goa7JVVUgMfZNhnK0xjrA+5+LEEr7paPk7ldYQtNdRj01acxktJfutWghVME172Nspza9zLNth3VE9UeC5Q3g6F3qSIlqgRkkiFASilHxye8A5K1wMnq9eus7l3iZe+mTOkQZbd4k3RTghcE0DL+UlZzPoV4ekmlK4SJVmXF4k/KvXdznfVz45HGKk7lUQrU1tmqSMJj0yo0zVcv/5LeL9G4gErAYkKCoZqmApiJpQVf4Iz6TthVif8+mJ9VFN7ULdq112h9IqRBtHdgrR1sVb9tSdjh/sSjVvM+5YH3zVDjkLCPt2wNeuHOBtRqU1OigopY1IhPti5McfPEMqFVuTlK2J5Uqe46j5B1E80WpTRlaiKCnC19/Y4dxj53ggrQtpB85xeVHxB5euctVmZMsVh2KZNt1gVgyj4ZDeYIC31DWIBvYRMTgiRkuiEQ5I2V2UTOMhYRU6oKk1y8450jTt2g/aAtTaU2wGrr3zitC6izZmbJW0U9V2uGXLwL3T8yoDsFgsUXVN/rw+LbL58PUI81bvE0LXm6GkfG8359UoZMFTuYrE9zExsExqIs0nhyMuDB0aSsaS8rFzU165uMvcOkqbYLQCNbhQ36Q0xy+8rgP+ybNzHhwZsp5ld37Ia8sVO70+BAelMjeGLZcwzHr0s/rcbbTu/jZNydxQj0+WYFDJ2AX+n5d3efnSNX7xw+d40NQBX4fkNxutPvGmYDKZHpkl9XZ04G71Zl0hWkr+EYVYPyj09JmUDSMqQojKgSq/d3XBG6WgSY43NA01VU0TCylpFJZGqKIntWmnnTctSqjz9NIjoaasXTOWP9hbUonBSD1sI5qA0YpBMaAnBZ9/bJsBijcJqVc+dGbKv7x0QFoJVVI1rlzX0EVBxVKJ4Spwfe6J8xI1Bmf7SHAk0XDdBc6kjrPTSZNl3mQrm6bFIbRpq5YEgdcq4R8/d4Vv7QcyY/j23pwL2yN6umLuDP0yQU1JtEqFw2tgsX+VZOMMkg5qv6S1Aq0MDfLYZi43A/MkQO4CNihRI9YrauTUMdAm1F1glVisWtRq0wJpOgVd7+VtDUCnEK3ZaI8EOjWOaBpPVCK7ovwvX/8+z4UBuU2xKrW5oG4MVnUUDqaryP1pRkJ9aIo2PSCt37JiiVbwWpLFwCwd8I1rO8xiPRc7tFA69cPMCHxsI+XRadbQygzqPPenGY8Penx3HonSVGsbyxSO3EbdDByb8nJN+tWmHS9iEkMvczfN9toz8LbFTerCXUHGC4uc3/j+K7xYOfK0Pvfq+3sFX5wK59oabkP0fX4JFw8KTOIYGWVY5GxNEqZG6bkam6pMZBldSwTsUkYFUmKt1CZjRxwjajJTfWDMTVGEXCxWE0DZltPJQe1BOG3IcKKVL03T7mC1U2cgiWlOqKkwJoHxlOVCiGrJqlq7ovFIU5lbphUbqjyY1rX8ECPz+aIbjSciDGRIsjUgsAL1vFoI35zdnCJzUykFSOm5GX/h0YcZEQhqMN4zYwVFwo9vTfne8ipR+9g7jcS1Jp4gkUBgC8Vlt0H0Y90pvjLC16/P+WcvXOeKDlFb92OoSdj1wvPzFWcmCYaKKPVUuOeD8NVZycp6nJbojV3QOb3gSShxJoJLqMQRSdeurbYCwdRM9hUZ/+i7rzI0EU+Cnjo0omZYffR+xy9/4JFatdYGsLcbn7WGHY43+yZJwmq1IsZYHwlwbF5ikIhBkGjYQPm5B+7jte9eZOYyjPSaCXDNeEHxDMsDPvPwfaTOUAksDg9ZNscqd0cih4Ll3orNrT77kz7feHGP/ZiRnoislH7I+fwDIx7pW0JRMlsVkC9ZukivTHgwyXjAVlyOvTtOzKQZJiaqYAIfyvokcnrE46raMuwb5V9fvM6/fmOfPTuECFmsMDEQ8URjeWEZ+PFJD6HuajfR0Q/Q80JFgmkQylJSZmkfq76eQdWYBGtAY2h4H3UZW2093V9JuEoF0dQl8G5M7s1/bbRYTbk/rI0zXXumbaxojCFJkpvg1HEL0e7cdpjEUUOktf8Uh1XPRzctX35ki61qBRQ1uCOgeMZxxVfOnuHLD5zDAGVQlovVCR5f4QqsLwlLeAXDc4sVqaanLJxyPiv58xe28fmCg70l5aqgFGVQ1L/fswt+6uxWM4D0Dg0EhiCmLqhp4HPnzpDp6VhMFOWNGPi1Z1/nX7w6Z0GfpIqkMYBaIgkqhmiUV3OPbxp6Y9NfqSbgbUVlFW+0GSNQMq5W9H2g54WeN2ReCVqCeKx6shjohwCxIokB5wPB1Awvo5E0lqSx6P7NYoGhwhtb3xt15rZeHWnXN8Z4ZDThEQvR5sohBPI8Zzwer8US0pRqm7Zza3Di+TOPn+H+6YA/emOHNw4PSMVyfmPMp89v8vFxn15sonLv69r+iWpfwFSGvHT8zquvMbMZvVJZpR7RFIkGpMTZkp//8AWmVclsMUc1xYg08G49hkAk8uHRgC+chd/fKYiSNFzE2s0JbcfZ+gEKthn4t+JDWz0eOzOuF9gA1G5EsThveK6I/Mb3L/FsAbntkYWIaw6di5I0PIeIaqQSWAKb0VJJk6ZGJRipoWpMM88BShuaXb5OgDWYUPK5B7f5semQUFQs8VR5JEZYKsSoVCXkWKoSni9h7i1GFSurerKodo2VNzdBc65529e5nlCcGDrWxhFte9fRQdymLo0IQIIlYarK57aGfHRr2JykU88d7Tef3w22WKPFH6mwAoep49nZgh8uKpY2w5pIEoWltYgom+WKL12Y8vFRxmJ3t/aZUhNTTTRdl5fRlCzm/MLDU0bZkt97bYeryZCghkFwWLUN5a62djaCF09GyZMp/NJj99FL6sNjJVKP94lQROGP9mf8xg/32fEOb8HgG1yiU+0uIwCth4wOMuwyJ2pdSjexVhQTImLrsY0uWJCTk2GUFKclj/fgZ7YTDOkRILpNFz1KJaDR8Heeucyzu4pT8FIfIIPWfBbTTZxtGpSaISFpmh7BnE5ET71erwv68jw/cdzxSZsbsQ20qypIoDuSeV2cc10f4Xr396AU3khSvvr6DWKYkCmU1pOEAVUayOKcz5zp8ZUH7sfPZx3ydxoQFmPEGEOPwM8/PuWR7Yx/8/J1Xp5HCnXNtJe6ochowBEYS8Gnzoz4hUce5px1BGrS6gDBBstSLf/v67v85htXibFX1yiOYSzHaQYCDJzhzHTApCcsy5JQljjbOKnm/I3bSdSaX6E+tiHEkebtboyggCNSqYHYNAKZRjOPK1mjrGVZds/x+FntJxSi1ZgYI3meMxwOb4tCqrTDTpox5+boCPt1rsVkMmF/f/9IhS3S5/kbK14Pjhj7OClYpAFvSraKJR+bOH7+ww/RN5Gdg/y219IqmrOODMPnJhM+8tSYP3ljl1cWM67nnryqEAlsDDMuTKc8uTnmoUFGDyWGkvnBrCa/9jJyN+Sfv7jD13aWBB2CtON/aNoIbioArPdHRh6f9OkBaZZiej2sRkarG8dK79zmXupdbbXJbMzJe203ZHucZVl5VFxzesepqwVIV2A7bXLvqX0Zrdtojw2+HWpZk1z15sAQ0x5OcPKS0jRle3u7e29rLSHrMbHwSFlyY75P6Qv6RM64gi+c3+KLD55lbIUyVoQ7TB2cSTDe1dYy5jyeFHxk2sOP03qEjxU2N0Zkri59S1Aq69nLZ4RVTq8yzJaB//PKa3x/lSGxj5OK3LSMrNOfeNtin3jPU1tj+rE+UBZrmxGFRwer3HKTcbOCPEiyJna7xUYQQ1TBC5S+niVeU479qdcXY23524Ehx4+gPlUh+v1+V+harVZHJo0cl9oK3twqt59mD2Ito/EYadhLMSo/c3abL547w37pKWLAidBzCWMLqUYkKuItpp42sdYR0jwo0c5393o9TOqoiCSqJHlJEoVFCklcNedyGWI6oDIRpwYrgi9K4jzHYFlmBiXjg71zvHS44HpfSdU3TTa2I+dI0wQtKphoCKJo8JwfD7jP1RgJ3QOXt1CjvBnEb/QdphmKfPMd1udfNwPQVSmDNL218ZYFkbIsu/jjtHM5jhiidsGzLOt6NFptum3xas1LmNuRvTVSiVKq1gf+NrxATCQV5VxiebCf8GBq2bb1lHpM3dHlXW136gn2EUvZZBfNIHKJWIHRoF9H2UGIqpS+7sqy0dccwyBk6kikRiPrvobAPJ9DEJIQCXZFFpf8+FaPL25Ztqo51jt6QZuhY/XhtPVwlICJSlbVQd85yfmLZ4f0/YpgDLTEVXmrRWtBrSFNmgGbrTq055IRm0EiERcjpShF4aksWPypJxGpcuRg2jdViHXFaIONNgW9e6lb5rKmnDdzUBpLaRKsN5joKW1NVT00lqIhiQr1BJnUKhmBXixqgqlabFSyoKTRYKNlOtnGJv16ar4oM6PkMRCMkvqUlYzQmOJESVVJogU1FBqRFVhSDnp9xssxEi3BzvnshSGfGyT0Y0WQPiY6slCRhQoThaTq4SVld1gysHP+3IPbPC4VooGo+rZKl3UstsKYHOMM6qq62VfbJqb62ElRi0RBQtW0HlSYCEHsqS6tPW9Ltbakpx3ldMvN3Ba3jDFd1nEPVAK8MMtLfueHPySP9ZGJwdXBWhZgryj53Rcv8/KNA2KsahZy0wpg+n2iRIIx5GaAGlePIUhT+tvbzJ3hB9euc6XIsVG5tnPAD/MDQLkWI9+4ccDF0hOTrG5/q2BlPZIrRezxTLXg6YN9vn9Y1lbA5GyHA372/g0+OLLEEFEj9cAvU+Gt4K1FTeCMzvmF85t8sgdW5nf1vARh4A2Dsk8RLHPvKHxCgVJKpJKAp8JT4gmULuFAa4vrYt0ItB5ztFeyWi27GlJLdL6jE3Vahk+WZeR53o3Sv7ujA4WojpUVvvbMy/SGfRJj0KB4BBcTXlst+YOLL/OR848yb2KCKsC8Dja4WNTH+tk04/JOwQe3ejy8MSAkPZ579XVWScqVRcmDScYk6/H09SVPbm/y2uEVrmjKDoaNaFktS1b7F/mz55+gQEnV8b3DFS8tr/Pkw0/wrVducEHHZL6e0T0xJT/70BaLV/a5tKpY2brB2QZPSs55PP/h+bN8OI2YmLNylkTN2z7SMgrsJilLY/mfvnOZDbcii5Ykq4tV/SSllziyJGFshZ54bggUtoeLENqmjzWNCCF2dSprbQdXHw8Hbnsq33A47NxFe5Lb3UjhI9+dHbLMUr782MNkRLAlCRnXo/J7P7jIFz7+QV68cgNcxuHhgvlCeGEmhLjkGW/YTgp6A8PFw5JHzk9J0gwbhe2NDfJ+n++99iJb57b47SvXeGkBj40M92djFtKjuHaNB8eP8H9feoZffPIhcJFxVJ4OOU8fLvjsmQfJSiENBkmEQZnhjVKYwIN+wV96Ypt/+sxrvBRTSrVM44oPDixfPn+Oh3TB0uVUJmNY9HHZ2z9dT1TpxRIT+8xcwq560uDwy6TGTVVJqJCwwkRTl9VdiXW9ehaWhjXKTS2Led6k++ZIX8xxC3FbFW5PcmtPX1kfgXtbaRS07sPyda4clG9fvc63Lr3Mjz10H1MgaFnDvMAfX36Vxx7aoozCV9/YZyNRyuWSflww6XmqrS2em6/Y1B5nJmPOp45HJgNWzTDP4BJ+8+kX2ZhskJWey1f2+I8+cJaziSdPBvzgyh4fPXOG789meDWcnw4weCpN+MGrF/mpB7e43055+o0rfGx7QBqVytSt+SbWA9ceM4G/9LHHeFAjD1cLvvLYBn/9k4/wxMQhqcFJj1QtxgSyXo8QAovFgsPDQ1arnICQ+mZYqSY1UtoeTdUc5Vj/K83xVBUmRJxP0ChYDSRan0OqGlFr8IlQ2Xp6ft0e6JsTEn0z2TbgY8UqX9TD044d63Rcbtu51Z71XRQFxhjm8zmbm5tvTpcTOiqGYqjEUBgYjDd44FrOQ0mfGAQbM9RFShUuHVQ8+uADvPDiFYbDCatFSdEfkNhI0h/z3Wcv8ZnNDZ7cnvDPLz/Ppx7+IDIvGA4DVeK4VioPDzI+ff8Gf3D5NT68OeH+vqVcFOzaAUsFJxnX8iUPb2xy/cYe07NneHmZc94NeTDJ+MODHbaSPg+N+piw6ngUtpnNMLDKkyPHX/7kfSTW8PDAMtAAoz467B2ZxJPnOTs7O91rGMtBWQfJ9fiyuuYQjWKjWTPxTd+HSMNHW0d+9QhC2k75NsTuz9tKeCVDJGaI5syWK0xcESWtU/67Oeq5PSmuKAqKoqAsy9seFUyTG8fmwBFVYaGRZ3b3+b3XrvKRrW2+c3WXa/k+T2xMeerMBhrqh/j951/jJx4+x+HLr+GqFAYZnsjVq1c5M3R86WyPfrlgpIbF4Qp3bgy2oiDywsWrfPHhLe4bOkpRPvbAGWwo6uarsmCMcu3Kq3z0ocdZXL/Bdn9ST9dTmGQDfnCwg/cLfmbjASotj5S/2wKQTTPS6Pno2BJRbPTNyJiboBRNJXE2mx0ZzhpVcWoYWUdmhUTa4Ww1b/NeiqiSxhVWc6ZqyHMhBZw1b3o6s+gdhMPe+07bnXNsb2+fKGMfVwhtGlpDVF6fL3hu5wabGxM+urmJKCwJWB8Yu0hlHD88KOg5w+O9Pgd+yeJgH9ccEr9AWCUJ45gzrBLc5ja9foKIkrMki32euzzj0YeH9Ii8eOB5cJzhVgv2D+csTI+rhxXToTJOMu7bmJDZuvC2iIbLN+aY1DLpJ4TdPbJg8eKPpG6DwYDJZNqdQ1gXFk62xokIBwcHR/L9bvepUjhha3ub1Lhm3tSdNOm/NVGgaK5yeTgnLBaghs3plMGgf1tc6U0VYv0mFw25ZTqdMhwOu2LS7S4tRE9Qg4rBNZ1EVmvOnzZHAySqFEbRaOlVUNiC/YNdYhUbFFSaUnE9pf/MxhTr6gn7OZF+EEIzCLyenF/bqNViweF8QcSiajESSU1g8+w2mBSDEtVT4si8RU3JfliiOwEvVcdVNMZw5swZTDNcrOu66FoPjlZy9/b2mpOJjj3smFAaOLs9pW/rVPqdUIhaPL5Ysbs3q6mHqWV7axt5E2v0ptfSatJoNOpOgj08POz4eLdU0a5CV5vHLNZHKLVzEaBO3bzUi5WgZAawBuN69IcbYLN6nK8aHI5EE4aTST3kywSswjCamn1kfD3PuTmQzapp8vH6qERHVY/8w1LXOeuJ+xZD1ox/tQpbyYDeuQ3SpvVQRNjY2Ki/787NbCUe6dJen+QHnMAioniM8V1pg4Zdxj3/Eqpo2T8siGqxxjIdj+4IPL/jcQDt8cQHBweoKvP5nI2NW4zT6Uobgm0OImtVL21ruPXBvvSpj0yqvVx9RQmCywYMXH1swvqhp0dSX3PzG0fa0i7r/7PUKF4zuJ2m69Jgjs7Pl6SdiwKunrg/cqAbG0eGgocQEOduVp5EuvhhffO0KGAL+R/lpirDtEdi3M0ezDtdgDuUttdjtZhTNpt22M9wSe+OUNO3NB+i5UoURUGe5yyXS4bD4T2+pZsPt1XCkzf8zneyhBCYz+cdmNNWgSeTyYkK4XFJ05TpdNqdoNtec/v37+T1t/T65XLZMenfjMKwLm9JIUSE6XTKzs4OMUbm8zlJkhzhYt7rm7uT106T9fa046/fyd+254Cvv5bnOTFGtra2bhk7tdfXngfSVheTJDlCZn2nJMbI/v5+d5+TyeQtAWRvKZ5pm0Ink0nXKDqbze5uhMA7KG/nmlp6ehsUHlegqqruuNhnraXf7zMcDt8VZVBVDg4OOgLScDh8U4jguLwlhVjX/rZ0WlXVkZz7R0He7FrbRiKOKVX7/dvpkH83NsxyueyU1Tn35vTHU+RtZTyt62h96Wq1YrFY3Jx58D5RjluZdX8Lmv7xCfHcYiHfbtHqnRJVpSgKDg8PO4BsOp2+LSV823dmjOmyDBFhsVjcea3jXZJbxRBvprBJkhwZs3T8Pe+2yHevxXvfxQ2qekeB7y2f2du9iLZjeGNjo7uQ/f39rj3s/SAt6+u4rM9VOn5P69nNcYVq/fLbfdjvhIQQ2Nvb69ZgNBp1BUm5Q0LvutzVnbW8vPF4zGw2Q0TY29tja2vrtjzMd0Na03naQ7mVQqxLv9/HOdf1qBhj6PV6ZFn2vgmg24xinVJ/HAZ4q9d6T1R9OKy7ntuHt7+/z+bm5tvmA9wrOV5vab/3d9jql6bpiZT6/RIftW6i7dTv9XpMp9NuZvXblbuOjtqHPp1Ou0paqxTvxbyqVtpFXDf7rVm9E4U4TZF4l7KFN7uvNt1f7756u0Hkcbln4XKrFIPBoMvl9/b23rOYoutsOsXft5NTfhSldRMtRyVJks4aHx9T/XbknudPk8mk87OtUryX2cdpbmtdId4vLuBOJITA7u5ud0a7c46NjY17mgbfU4VoTfTm5ib9fh9jDN57bty40RV7eBcXYX120rocP7j2/S5tt/bu7m4XM7QZXluBvlfyjiAsbR/nYDDoIv29vT0ODw/fdZjbOXfqDno/pcdvJnmes7u72ylxr9djY2Ojm2R3L+UdhdzG43FX96BhbrcVwHdL2uOMj8v6wK33q7RFtv39/e610WjE5ubmLTGWu5V3DGFpL7Z1HQcHB8QYWS6XVFXFdDp9Vwo+xhicc91A1la898T49qny77S0aeX6gLbxeHyiweZeP793HJRvwautra3O37Uczbb+8U5LlmWnglN3ike8W9KmxavVip2dnW72eDsWueU1vJOb6F2r0iRJwtbWVpeWtrT+Nmp+p4piqtqBS8fl/VR7afGF/f39Iy4iyzI2NzdPbcx9J+SOWNf38qZZq46u8zIHg/p023fCN8YY2dnZORG7JEnSMcjfS1FVFosFi8Wic23GGEaj0VtiO90LeU8UgmaRDg8PO6oXjb8fj8f1jId7mFu3xJGWDtfdvAhnzpx5z4pVLQtruVxSFEVXd8myjPF4/J5UVd9VhThN2qaWdX/e8gBbKPxe7JDVasXe3l7ng9sWgvF4/I7xQo/L+qOuqor5fH6Ert9ahTYQfy/kPVeI9kHN5/POZLY7JU1TBoPBPbEYIQRu3LjRsajbz82yjK2trXfFLLcA03K57Aa9tzIYDBgOh0cg6PdC3hcK0Yr3vjt1Zv2yjDGdxbibNHFvb++I22gf/NmzZ+/pQhwv+7cD3FrGOsfS8nWOxXsdz7yvFKJ9kN77bhetWwyaqPs4afVOH+JqtToSwbcyGo1O0P3v5h5Yg8dbJTge0PZ6vSPNT+8XeV8pRCvtJbXnS7Q8C9aOR7TWdoSVJEnuyHKEELoWgnWx1tatenfplmKM3Qim9Qaj9WMQe70eg8Hgjs8jebflfasQx01uURQdyrluNbrDw5rTatI07Xbd+nu0tzmbzU5txN3c3Dx6oMmxXXsa+6vFDsqy7L7WLcF6Cb7f73dznd5LJtmbyftSIdZl/Uyu1p20ZriFn4/cUHsg6dr5Vu1XW33d2dk54m7aAHZra+vEe7WAWVsy99538zvbyuM6W3tdSbMs6xp23q8KcFze9wpxXI4f2ra+O2+Hdq4zqE4jx7QLuN5KEELogKLTSujHmVTtXK5er/e+CRLfqvzIKcTtpKqqI6a7XdAjN3yMG3mr278VN7F9rR3c1X4dHyL+oyp/qhRi3XqwFuStf7VuZl0h1g8zbaUtm7dWpY1TrLXd14/a7r8T+VOpENwmCDxuIW61qMf//lbvfTzG+VGXP1UK8f/L3cv/B0ZrUUkKONRwAAAAAElFTkSuQmCC',
                submitLoading: true,
                loginToken: 1,
            };
        },
        onLoad() {
            this.canIUse = uni.canIUse('button.open-type.getUserInfo');

            this.loginRequest();
        },
        methods: {
            loginRequest() {
                this.submitLoading = true;

                uni.login({
                    success: res => {
                        console.log(res.code)
                        this.loginCode = res.code;
                    },
                    fail: err => {
                        console.log(err);
                    },
                });
            },
            getLogin(e) {
                this.bindCheckCacheUserInfo(e);
            },
            bindCheckCacheUserInfo(e) {
                const that = this;
                this.submitLoading = false;
                if (!e.detail.encryptedData) {
                    uni.showToast({
                        title: '微信信息获取失败',
                        icon: 'none',
                        duration: 2000,
                    });
                    return
                }
                uni.setStorage({
                    key: 'userDetail',
                    data: e.detail,
                    success: res => {
                        e.detail.code = this.loginCode;
                        uni.setStorageSync('userDetail', e.detail);
                        uni.removeStorageSync('loginToken');
                        that.checkSessionAndLogin();
                    },
                    fail: err => {
                        console.log(err);
                        this.submitLoading = true;
                        uni.showToast({
                            title: '微信信息获取失败',
                            icon: 'none',
                            duration: 2000,
                        });
                    },
                });
            },
            checkSessionAndLogin() {
                const thisOpenId = uni.getStorageSync('loginToken');
                if (thisOpenId) {
                    uni.checkSession({
                        success() {
                            uni.reLaunch({
                                url: '/pages/index/index',
                            });
                        },
                        fail() {
                            uni.removeStorageSync('loginToken');
                            this.checkSessionAndLogin();
                        },
                    });
                } else {
                    this.loginAndGetOpenid();
                }
            },
            async loginAndGetOpenid() {
                const that = this;
                that.submitLoading = false;
                const userDetail = uni.getStorageSync('userDetail');
                this.$tools.dd(userDetail)

                if (!userDetail.code) {
                    uni.showToast({
                        title: '微信信息获取失败',
                        icon: 'none',
                        duration: 2000,
                    });
                    this.loginRequest()
                    return;
                }

                const paramForm = {
                    code: userDetail.code,
                    encryptedData: userDetail.encryptedData,
                    iv: userDetail.iv,
                };

                console.log(JSON.stringify(paramForm))
                const result = await wxLoginApi(paramForm);
                that.$tools.dd('登录请求返回数据', result);

                const data = result.data
                if (data.code != 0) {
                    uni.showToast({
                        title: data.data
                    })
                    this.loginRequest()
                    return
                }

                uni.setStorageSync('userToken', data.data.token);
                userInfoApi()

                uni.showToast({
                    title: '登录成功',
                    icon: 'none',
                    duration: 2000,
                    success() {
                        uni.reLaunch({
                            url: '/pages/index/index',
                        });
                    },
                });
            },
        },
    };
</script>
