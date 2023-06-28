# delete discord messages (api v9)
### Step 1: Open developer console on discord tab
### Step 2: Edit the following values and paste into console
```js
// your username
let my_user_name = "";

// get your request headers from a request and paste here
let my_headers = {};

// the channel that you want to delete your messages from
let channel_id = "";

// id of the message which you want to delete all previous messages that comes before it
let fetched_next_before_point = "";
```
### Step 3: Copy paste the following functions into console
```js
async function fetch_and_delete_50(before_point, stage_num) {
	const data_from_fetch = await fetch_next_50(before_point);
	let fetched_message_list = data_from_fetch.data;
	fetched_next_before_point = data_from_fetch.next_before_point;
	console.log("****************************************************************************");
	console.log(`****************************** STAGE ${stage_num} ******************************`);
	console.log("****************************************************************************");
	await delete_50_messages(fetched_message_list);
}

async function fetch_next_50(before_point) {
	let response = await fetch(
		`https://discord.com/api/v9/channels/${channel_id}/messages?before=${before_point}&limit=50`,
		{
			headers: { ...my_headers },
			referrer: `https://discord.com/channels/@me/${channel_id}`,
			referrerPolicy: "strict-origin-when-cross-origin",
			body: null,
			method: "GET",
			mode: "cors",
			credentials: "include",
		}
	);
	let data = await response.json();

	return {
		data: data,
		next_before_point: data.slice(-1)[0].id,
	};
}

async function wait_until_1_msg_gets_deleted(message_id) {
	return await new Promise((resolve) => {
		const interval = setInterval(() => {
			fetch(`https://discord.com/api/v9/channels/${channel_id}/messages/${message_id}`, {
				headers: { ...my_headers },
				referrer: `https://discord.com/channels/@me/${channel_id}`,
				referrerPolicy: "strict-origin-when-cross-origin",
				body: null,
				method: "DELETE",
				mode: "cors",
				credentials: "include",
			})
				.then(() => {
					resolve("foo");
					clearInterval(interval);
				})
				.catch((e) => {
					console.error(e);
				});
		}, 1100);
	});
}

async function delete_50_messages(fetched_message_list) {
	for (let i = 0; i < 50; i++) {
		try {
			let message = fetched_message_list[i];
			if (message.author.username === my_user_name) {
				await wait_until_1_msg_gets_deleted(message.id);
				console.log(`Deleted [${i}]: `);
			}
		} catch (error) {}
	}
}

function fake_delay(amount) {
	for (let i = 0; i < amount; i++) {
		for (let i = 0; i < 999999999; i++) {}
	}
}

function avoid_rate_limit(num) {
	fake_delay(10);
	if (num % 3 == 0) fake_delay(30);
	if (num % 8 == 0) fake_delay(90);
	if (num % 150 == 0) fake_delay(150);
	if (num % 300 == 0) fake_delay(300);
	if (num % 500 == 0) fake_delay(500);
}
```
### Step 4: Copy paste the following code into console in order to start deleting
```js
let num = 1;

// 500 means that the code will iterate through 500*50 messages and delete the ones you sent
while (num <= 500) {
	avoid_rate_limit(num);

	await fetch_and_delete_50(fetched_next_before_point, num);
	num++;
}
```
