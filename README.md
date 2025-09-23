<h1 align="center">Hi 👋, I’m Arghadeep Kar</h1>  
<h4 align="center">Computer Science Student • Passionate about Web Dev & AI • India 🇮🇳</h4>  

---  

## ✨ My Journey

- 🧠 Strong foundation in **Python**, **C++**, **Java**, and **C programming**
- 🎨 Enthusiastic about **UI/UX design**, using tools like **Figma, Adobe XD, Bootstrap, and Tailwind CSS**  
- 🚀 Developing **AI-powered** solutions and working with **MERN/Django frameworks**   
- 💡 Interested in building **intelligent apps**, **creative interfaces**, and **interactive digital experiences**  

---  


## 💬 Connect

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2)](www.linkedin.com/in/arghadeep-kar-75451728b)
[![Gmail](https://img.shields.io/badge/Gmail-D14836?logo=gmail&logoColor=white)](mailto:arghadeepkar2020@gmail.com)
  


---

## 🛠 Tech Stack & Tools  

**Frontend**  
💻 ![React](https://img.shields.io/badge/React-20232A?logo=react&logoColor=61DAFB) 
![Bootstrap](https://img.shields.io/badge/Bootstrap-7952B3?logo=bootstrap&logoColor=white) 
![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-38B2AC?logo=tailwind-css&logoColor=white)  
🎨 ![Figma](https://img.shields.io/badge/Figma-F24E1E?logo=figma&logoColor=white) 
![Adobe XD](https://img.shields.io/badge/Adobe_XD-FF61F6?logo=adobexd&logoColor=white)  

**Backend**  
⚙️ ![Node.js](https://img.shields.io/badge/Node.js-339933?logo=node-dot-js&logoColor=white) 
![Django](https://img.shields.io/badge/Django-092E20?logo=django&logoColor=white)  

**AI / ML**  
🤖 ![Python](https://img.shields.io/badge/Python-3776AB?logo=python&logoColor=white) 
![TensorFlow](https://img.shields.io/badge/TensorFlow-FF6F00?logo=tensorflow&logoColor=white)  




---



## 📂 Featured Projects  

✨ Coming soon... *(something exciting is on the way!)*  

---

## 📊 GitHub Stats  

import React, { useEffect, useState } from 'react';

const graphqlQuery = (Arghadeepkar85) => ({
  query: `query($login: String!) {
    user(login: $login) {
      name
      login
      avatarUrl
      followers { totalCount }
      repositories(privacy: PUBLIC, first: 100, ownerAffiliations: OWNER) {
        totalCount
        nodes {
          name
          stargazerCount
          primaryLanguage { name color }
          languages(first: 10) { edges { size node { name } } }
        }
      }
      contributionsCollection {
        contributionCalendar {
          totalContributions
        }
      }
    }
  }`,
  variables: { login: Arghadeepkar85 }
});

export default function GitHubStatsWidget({ username = 'Arghadeepkar85' }) {
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  const [data, setData] = useState(null);

  useEffect(() => {
    const token = process.env.REACT_APP_GITHUB_TOKEN;
    if (!token) {
      setError('Missing REACT_APP_GITHUB_TOKEN in environment. See component comments.');
      setLoading(false);
      return;
    }

    fetch('https://api.github.com/graphql', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        Authorization: `Bearer ${token}`,
      },
      body: JSON.stringify(graphqlQuery(username)),
    })
      .then((res) => res.json())
      .then((json) => {
        if (json.errors) throw new Error(json.errors.map(e => e.message).join(', '));
        const user = json.data.user;
        // compute top languages by summing sizes from languages edges
        const langMap = new Map();
        if (user.repositories && user.repositories.nodes) {
          user.repositories.nodes.forEach(repo => {
            if (!repo.languages) return;
            repo.languages.edges.forEach(edge => {
              const name = edge.node.name;
              const size = edge.size || 0;
              langMap.set(name, (langMap.get(name) || 0) + size);
            });
          });
        }
        const topLanguages = Array.from(langMap.entries())
          .map(([name, size]) => ({ name, size }))
          .sort((a, b) => b.size - a.size)
          .slice(0, 8);

        setData({
          name: user.name || user.login,
          login: user.login,
          avatarUrl: user.avatarUrl,
          followers: user.followers.totalCount,
          publicRepos: user.repositories.totalCount,
          totalContributions: user.contributionsCollection.contributionCalendar.totalContributions,
          topLanguages,
          raw: user,
        });
        setLoading(false);
      })
      .catch((err) => {
        console.error(err);
        setError(err.message || 'Failed to fetch GitHub data');
        setLoading(false);
      });
  }, [username]);

  if (loading) return <div className="p-4">Loading GitHub stats...</div>;
  if (error) return <div className="p-4 text-red-600">Error: {error}</div>;
  if (!data) return null;

  return (
    <div className="max-w-3xl mx-auto p-4">
      <div className="flex items-center gap-4 mb-4">
        <img src={data.avatarUrl} alt="avatar" className="w-16 h-16 rounded-full" />
        <div>
          <h2 className="text-xl font-semibold">{data.name} <span className="text-sm text-gray-500">@{data.login}</span></h2>
          <div className="text-sm text-gray-600">{data.followers} followers · {data.publicRepos} public repos</div>
        </div>
      </div>

      <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
        <Card title="Contributions (year)" value={data.totalContributions} />
        <Card title="Public Repos" value={data.publicRepos} />
        <Card title="Followers" value={data.followers} />
      </div>

      <div className="mt-6">
        <h3 className="text-lg font-semibold mb-2">Top languages</h3>
        <div className="grid grid-cols-1 sm:grid-cols-2 gap-3">
          {data.topLanguages.length === 0 && <div className="text-sm text-gray-500">No language data available.</div>}
          {data.topLanguages.map((lang) => (
            <LanguageBar key={lang.name} name={lang.name} size={lang.size} total={data.topLanguages.reduce((s, l) => s + l.size, 0)} />
          ))}
        </div>
      </div>

      <div className="mt-6 text-sm text-gray-500">This component fetches data live from the GitHub API, so numbers update automatically when GitHub data changes.</div>
    </div>
  );
}

function Card({ title, value }) {
  return (
    <div className="p-4 rounded-2xl shadow-md bg-white dark:bg-gray-800">
      <div className="text-sm text-gray-500">{title}</div>
      <div className="text-2xl font-bold">{value}</div>
    </div>
  );
}

function LanguageBar({ name, size, total }) {
  const pct = total > 0 ? Math.round((size / total) * 100) : 0;
  return (
    <div className="p-3 rounded-xl border">
      <div className="flex justify-between text-sm mb-1">
        <div className="font-medium">{name}</div>
        <div className="text-gray-500">{pct}%</div>
      </div>
      <div className="w-full h-3 bg-gray-200 rounded-full overflow-hidden">
        <div style={{ width: `${pct}%` }} className="h-full rounded-full" />
      </div>
    </div>
  );
}






✨ *"Turning ideas into reality through code & creativity"* ✨
