---
title: "随机数_随机字符串"
publishDate: 2018-11-05 14:26:00 +0800
date: 2018-11-05 14:14:08 +0800
categories: 随机数_随机字符串
position: problem
---

---

<div id="toc"></div>

```c#

        /// <summary>
        /// 获取随机字符串
        /// </summary>
        /// <param name="Length">长度</param>
        /// <returns></returns>
        public static string GenerateRandom(int Length)
        {
            long tick = DateTime.Now.Ticks;
            var seed = (int)(tick & 0xffffffffL) | (int)(tick >> 32);
            //var seed = GetRandomSeed();
            Random rd = new Random(seed);

            return GenerateRandom(rd, Length);
        }
        /// <summary>
        /// 获取随机字符串
        /// </summary>
        /// <param name="Length">长度</param>
        /// <returns></returns>
        public static string GenerateRandom(Random rd, int Length)
        {
            string strSep = ",";
            char[] chrSep = strSep.ToCharArray();
            string strChar = "0,1,2,3,4,5,6,7,8,9,a,b,c,d,e,f,g,h,i,j,k,l,m,n,o,p,q,r,s,t,u,v,w,x,y,z"
         + ",A,B,C,D,E,F,G,H,I,J,K,L,M,N,O,P,Q,R,S,T,U,V,W,X,Y,Z";
            string[] aryChar = strChar.Split(chrSep, strChar.Length);

            StringBuilder newRandom = new StringBuilder(52);
            for (int i = 0; i < Length; i++)
            {
                newRandom.Append(aryChar[rd.Next(aryChar.Length)]);
            }
            return newRandom.ToString();
        }
        /// <summary>
        /// 产生随机种子
        /// </summary>
        /// <returns></returns>
        static int GetRandomSeed()
        {
            byte[] bytes = new byte[4];
            System.Security.Cryptography.RNGCryptoServiceProvider rng = new System.Security.Cryptography.RNGCryptoServiceProvider();
            rng.GetBytes(bytes);
            return BitConverter.ToInt32(bytes, 0);
        }
```