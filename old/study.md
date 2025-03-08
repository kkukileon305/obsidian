# useActionState with form

## Form.tsx

```
"use client";  
  
import { useActionState } from "react";  
import { signIn, signInWithButton } from "@/app/signin/actions";  
  
const SignUpForm = () => {  
  const [state, formAction, isPending] = useActionState(signIn, {  
    email: "",  
    password: "",  
  });  
  console.log(isPending);  
  
  return (  
    <form action={formAction} className="flex flex-col">  
      <label>email</label>  
      <input type="email" name="email" className="bg-red-400" />  
  
      <label>pw</label>  
      <input type="password" name="password" className="bg-red-400" />  
  
      <button className="bg-gray-400 disabled:bg-blue-400" disabled={isPending}>  
        Submit  
      </button>  
      <button type="button" onClick={() => signInWithButton(123)}>  
        test  
      </button>  
      {JSON.stringify(state)}  
    </form>  
  );};  
  
export default SignUpForm;
```

## actions.ts

```
"use server";  
  
type SignInState = {  
  email: string;  
  password: string;  
};  
  
export const signIn = async (  
  previousState: SignInState,  
  formData: FormData  
): Promise<SignInState> => {
  // api 생성없이 api 호출 가능
  const email = formData.get("email")?.toString() || "";  
  const password = formData.get("password")?.toString() || "";  
  
  await new Promise((res) => setTimeout(res, 2000));  
  
  return {  
    email,  
    password,
  };
};
```

- state를 입력값 대신에 에러상태만 저장해도 좋을듯

---

## actions.ts

```
export async function insertMessage(  
  previousError: InsertPreviousError,  
  formData: FormData  
): Promise<InsertPreviousError> {  
  const name = formData.get("name")?.toString() || null;  
  const content = formData.get("content")?.toString() || "";  
  const uuid = formData.get("uuid")?.toString();  
  
  if (content === "") {  
    return {  
      isSuccess: false,  
      errors: {  
        content: "content empty",  
      },    };  }  
  try {  
    const sb = await createClient();  
    const profile = uuid  
      ? await sb.from("profiles").select("*").eq("uuid", uuid).single()  
      : null;  
  
    const profileID = profile?.data?.id;  
  
    await sb.from("messages").insert({  
      content,  
      name,  
      profile_id: profileID,  
    });  
    revalidatePath("/messages");  
  } catch (e) {  
    return {  
      isSuccess: false,  
      errors: {  
        etc: JSON.stringify(e),  
      },    };  }  
      
  redirect("/post-message/success");  
}
```

이유는 모르겠지만 try catch 안에서 redirect를 쓰면 안됨

![[Pasted image 20241103054058.png]]

내부적으로 __에러를 발생시키는 방식__ 으로 redirect가 작동한다고 함 그래서 try catch 안에서 쓰면 안됨

---
## 웹 푸쉬

1. **Service Worker Registration:  
    **Register a service worker using `navigator.serviceWorker.register()`.
2. **Subscription Process**:

- Request notification permission with`Notification.requestPermission()`.
- If granted, subscribe to push notifications with `registration.pushManager.subscribe()`.
- This returns a `PushSubscription` object containing an endpoint URL and encryption keys.

3. **Storing the Subscription**:

- Send the `PushSubscription` object to the server for storing, enabling the server to send notifications to the correct endpoints.

