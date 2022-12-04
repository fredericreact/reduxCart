# Redux refresher

```javascript
npm install react-redux

npm install @reduxjs/toolkit
```
1. Add state slice


```javascript
import {createSlice} from '@reduxjs/toolkit';
 
const uiSlice = createSlice({
    name: 'ui',
    initialState: {cartIsVisible: false},
    reducers: {
        toggle(state) {
            state.cartIsVisible = !state.cartIsVisible;
        }
    }
});
 
export const uiActions = uiSlice.actions;
export default uiSlice;
```
2. Create store 

```javascript
import {configureStore} from '@reduxjs/toolkit';
import uiSlice from './ui-slice';
 
const store = configureStore({
    reducer: {ui: uiSlice.reducer}
});
 
export default store;
```
3. Provide store 

```javascript
import ReactDOM from 'react-dom/client';
 
import './index.css';
import App from './App';
 
import {Provider} from 'react-redux'
import store from './store/index';
 
const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<Provider store={store}><App /></Provider>);
```
4. Using Redux Data in React Components

Dispatch

```javascript
import classes from './CartButton.module.css';
import {uiActions} from '../../store/ui-slice';
import {useDispatch} from 'react-redux'
 
const CartButton = (props) => {
  const dispatch = useDispatch()
  const toggleCartHandler = () => {
    dispatch(uiActions.toggle())
  }
 
  return (
    <button className={classes.button} onClick={toggleCartHandler}>
      <span>My Cart</span>
      <span className={classes.badge}>1</span>
    </button>
  );
};
 
export default CartButton;
```
Get state

```javascript
import Cart from './components/Cart/Cart';
import Layout from './components/Layout/Layout';
import Products from './components/Shop/Products';
import {useSelector} from 'react-redux'
 
function App() {
  const showCart = useSelector(state => state.ui.cartIsVisible)
 
  return (
    <Layout>
      {showCart && <Cart />}
      <Products />
    </Layout>
  );
}
 
export default App;
```
1. Add state slice

```javascript
import {createSlice} from '@reduxjs/toolkit';
 
const cartSlice = createSlice ({
    name: 'cart',
    initialState: {
        items: [],
        totalQuantity: 0,
    },
    reducers: {
        addItemToCart(state, action) {
            const newItem = action.payload;
            const existingItem = state.items.find((item) => item.id === newItem.id);
            state.totalQuantity++;
            if (!existingItem) {
                state.items.push({
                    id: newItem.id,
                    price: newItem.price,
                    quantity: 1,
                    totalPrice: newItem.price,
                    name: newItem.title
                })
            } else {
                existingItem.quantity = existingItem.quantity + 1;
                existingItem.totalPrice = existingItem.totalPrice + newItem.price;
            }
 
        },
        removeItemFromCart(state,action) {
            const id = action.payload;
            const existingItem = state.items.find((item) => item.id === id);
            state.totalQuantity--;
 
            if (existingItem.quantity ===1) {
                state.items = state.items.filter(item => item.id !==id);
            } else {
                existingItem.quantity--;
                existingItem.totalPrice = existingItem.totalPrice - existingItem.price
            }
        }
    }
})
 
export const cartActions = cartSlice.actions
 
export default cartSlice
```
2. Update store

```javascript
import {configureStore} from '@reduxjs/toolkit';
import uiSlice from './ui-slice';
import cartSlice from './cart-slice';
 
const store = configureStore({
    reducer: {ui: uiSlice.reducer, cart:cartSlice.reducer}
});
 
export default store;
```
3. Using Redux Data in React Components

Dispatch

```javascript
import Card from '../UI/Card';
import classes from './ProductItem.module.css';
import { useDispatch } from 'react-redux';
import {cartActions} from '../../store/cart-slice'
 
const ProductItem = (props) => {
  const dispatch = useDispatch();
  const { title, price, description, id } = props;
 
  const addToCartHandler = () => {
    dispatch(cartActions.addItemToCart({
      id:id,
      title,
      price,
    }));
  }
 
  return (
    <li className={classes.item}>
      <Card>
        <header>
          <h3>{title}</h3>
          <div className={classes.price}>${price.toFixed(2)}</div>
        </header>
        <p>{description}</p>
        <div className={classes.actions}>
          <button onClick={addToCartHandler}>Add to Cart</button>
        </div>
      </Card>
    </li>
  );
};
 
export default ProductItem;
```
Get state

```javascript
import classes from './CartButton.module.css';
import {uiActions} from '../../store/ui-slice';
import {useDispatch, useSelector} from 'react-redux'
 
const CartButton = (props) => {
  const dispatch = useDispatch()
  const cartQuantity = useSelector(state=>state.cart.totalQuantity)
  const toggleCartHandler = () => {
    dispatch(uiActions.toggle())
  }
 
  return (
    <button className={classes.button} onClick={toggleCartHandler}>
      <span>My Cart</span>
      <span className={classes.badge}>{cartQuantity}</span>
    </button>
  );
};
 
export default CartButton;
```
Get state 

```javascript
import Card from '../UI/Card';
import classes from './Cart.module.css';
import CartItem from './CartItem';
import { useSelector } from 'react-redux';
 
const Cart = (props) => {
  const cartItems = useSelector ((state)=>state.cart.items)
  return (
    <Card className={classes.cart}>
      <h2>Your Shopping Cart</h2>
      <ul>
        {cartItems.map((item) => (
          <CartItem
          key={item.id}
          item={{
            id: item.id,
            title: item.name,
            quantity: item.quantity,
            total: item.totalPrice,
            price: item.price}}
        />
        ))}
      </ul>
    </Card>
  );
};
 
export default Cart;
```
Dispatch

```javascript
import classes from './CartItem.module.css';
import {useDispatch} from 'react-redux';
import {cartActions} from '../../store/cart-slice'
 
const CartItem = (props) => {
  const dispatch = useDispatch();
  const { title, quantity, total, price, id } = props.item;
 
  const removeItemHandler = () => {
    dispatch(cartActions.removeItemFromCart(id))
  };
  const additemHandler = () => {
    dispatch(
      cartActions.addItemToCart({
        id,
        title,
        price,
      })
    )
  };
 
  return (
    <li className={classes.item}>
      <header>
        <h3>{title}</h3>
        <div className={classes.price}>
          ${total.toFixed(2)}{' '}
          <span className={classes.itemprice}>(${price.toFixed(2)}/item)</span>
        </div>
      </header>
      <div className={classes.details}>
        <div className={classes.quantity}>
          x <span>{quantity}</span>
        </div>
        <div className={classes.actions}>
          <button onClick={removeItemHandler}>-</button>
          <button onClick={additemHandler}>+</button>
        </div>
      </div>
    </li>
  );
};
 
export default CartItem;
```
# Redux and side effect (async code) 

## Option 1 : handle side effect inside our component

Send HTTP request

```javascript
import Cart from './components/Cart/Cart';
import Layout from './components/Layout/Layout';
import Products from './components/Shop/Products';
import {useSelector} from 'react-redux'
import {useEffect} from 'react'
 
function App() {
  const showCart = useSelector(state => state.ui.cartIsVisible)
  const cart = useSelector((state) => state.cart);
 
  useEffect(()=>{
    fetch('https://react-http-26861-default-rtdb.firebaseio.com/cart.json', {
      method:'PUT',
      body: JSON.stringify(cart),
    })
  },[cart])
 
  return (
    <Layout>
      {showCart && <Cart />}
      <Products />
    </Layout>
  );
}
 
export default App;
```
Handle http request status and errors

```javascript
import {createSlice} from '@reduxjs/toolkit';
 
const uiSlice = createSlice({
    name: 'ui',
    initialState: {cartIsVisible: false, notification:null},
    reducers: {
        toggle(state) {
            state.cartIsVisible = !state.cartIsVisible;
        },
        showNotification(state, action) {
            state.notification = {
                status: action.payload.status,
                title: action.payload.title,
                message: action.payload.message,
            }
        }
 
    }
});
 
export const uiActions = uiSlice.actions;
export default uiSlice;
```

```javascript
import Cart from './components/Cart/Cart';
import Layout from './components/Layout/Layout';
import Products from './components/Shop/Products';
import {useSelector, useDispatch} from 'react-redux'
import {useEffect, Fragment} from 'react'
import {uiActions} from './store/ui-slice'
import Notification from './components/UI/Notification';
 
let isInitial = true
 
function App() {
  const dispatch = useDispatch();
  const showCart = useSelector(state => state.ui.cartIsVisible)
  const cart = useSelector((state) => state.cart);
  const notification = useSelector(state => state.ui.notification)
  useEffect(()=>{
    const sendCartData = async() => {
      dispatch(
        uiActions.showNotification({
          status: 'pending',
          title: 'Sending...',
          message: 'Sending cart data',
        })
      )
      const response = await fetch(
      'https://react-http-26861-default-rtdb.firebaseio.com/cart.json',
      {
        method:'PUT',
        body: JSON.stringify(cart),
      })
 
      if (!response.ok) {
        throw new Error('Sending cart data failed');
      }
 
 
      dispatch(
        uiActions.showNotification({
          status: 'success',
          title: 'Success!',
          message: 'Sent cart data successfully',
        })
      )
 
 
 
 
    }
 
    if (isInitial) {
      isInitial = false;
      return;
    }
 
    sendCartData().catch(error => {
      dispatch(
        uiActions.showNotification({
          status: 'error',
          title: 'Error!',
          message: 'Sent cart data failed',
        })
      )
    })
   
  },[cart, dispatch])
 
  return (
    <Fragment>
    {notification &&
    (<Notification
      status={notification.status}
      title={notification.title}
      message={notification.message}
    />)}
    <Layout>
      {showCart && <Cart />}
      <Products />
    </Layout>
    </Fragment>
  );
}
 
export default App;
```

## Option 2 : handle side effect using action creators

Thunks : a function that delays an action until later until something else finished

So we could write an action creator as a thunk so it will not return action object immediately, it will return another function which eventually returns the action


```javascript
import {createSlice} from '@reduxjs/toolkit';
import {uiActions} from './ui-slice'
 
const cartSlice = createSlice ({
    name: 'cart',
    initialState: {
        items: [],
        totalQuantity: 0,
    },
    reducers: {
        addItemToCart(state, action) {
            const newItem = action.payload;
            const existingItem = state.items.find((item) => item.id === newItem.id);
            state.totalQuantity++;
            if (!existingItem) {
                state.items.push({
                    id: newItem.id,
                    price: newItem.price,
                    quantity: 1,
                    totalPrice: newItem.price,
                    name: newItem.title
                })
            } else {
                existingItem.quantity = existingItem.quantity + 1;
                existingItem.totalPrice = existingItem.totalPrice + newItem.price;
            }
 
        },
        removeItemFromCart(state,action) {
            const id = action.payload;
            const existingItem = state.items.find((item) => item.id === id);
            state.totalQuantity--;
 
            if (existingItem.quantity ===1) {
                state.items = state.items.filter(item => item.id !==id);
            } else {
                existingItem.quantity--;
                existingItem.totalPrice = existingItem.totalPrice - existingItem.price
            }
        }
    }
})
 
export const sendCartData = (cart) => {
    return async (dispatch) => {
        dispatch(
            uiActions.showNotification({
              status: 'pending',
              title: 'Sending...',
              message: 'Sending cart data',
            })
          )
 
          const sendRequest = async () => {
            const response = await fetch(
                'https://react-http-26861-default-rtdb.firebaseio.com/cart.json',
                {
                  method:'PUT',
                  body: JSON.stringify(cart),
                })
         
                if (!response.ok) {
                  throw new Error('Sending cart data failed');
                }
          }
 
          try {
            await sendRequest();
 
            dispatch(
                uiActions.showNotification({
                  status: 'success',
                  title: 'Success!',
                  message: 'Sent cart data successfully',
                })
              )
          } catch (error) {
            dispatch(
                uiActions.showNotification({
                  status: 'error',
                  title: 'Error!',
                  message: 'Sent cart data failed',
                })
              )
          }
         
    }
}
 
export const cartActions = cartSlice.actions
 
export default cartSlice
```

```javascript
import Cart from './components/Cart/Cart';
import Layout from './components/Layout/Layout';
import Products from './components/Shop/Products';
import {useSelector, useDispatch} from 'react-redux'
import {useEffect, Fragment} from 'react'
import Notification from './components/UI/Notification';
import {sendCartData} from './store/cart-slice'
 
let isInitial = true
 
function App() {
  const dispatch = useDispatch();
  const showCart = useSelector(state => state.ui.cartIsVisible)
  const cart = useSelector((state) => state.cart);
  const notification = useSelector(state => state.ui.notification)
  useEffect(()=>{
 
    if (isInitial) {
      isInitial = false;
      return;
    }
 
  dispatch(sendCartData(cart))
   
  },[cart, dispatch])
 
  return (
    <Fragment>
    {notification &&
    (<Notification
      status={notification.status}
      title={notification.title}
      message={notification.message}
    />)}
    <Layout>
      {showCart && <Cart />}
      <Products />
    </Layout>
    </Fragment>
  );
}
 
export default App;
```
Fetch Data

```javascript
import {createSlice} from '@reduxjs/toolkit';
 
const cartSlice = createSlice ({
    name: 'cart',
    initialState: {
        items: [],
        totalQuantity: 0,
        changed: false
    },
    reducers: {
        replaceCart(state,action) {
            state.totalQuantity = action.payload.totalQuantity;
            state.items = action.payload.items
        },
        addItemToCart(state, action) {
            const newItem = action.payload;
            const existingItem = state.items.find((item) => item.id === newItem.id);
            state.changed = true;
            state.totalQuantity++;
            if (!existingItem) {
                state.items.push({
                    id: newItem.id,
                    price: newItem.price,
                    quantity: 1,
                    totalPrice: newItem.price,
                    name: newItem.title
                })
            } else {
                existingItem.quantity = existingItem.quantity + 1;
                existingItem.totalPrice = existingItem.totalPrice + newItem.price;
            }
 
        },
        removeItemFromCart(state,action) {
            const id = action.payload;
            const existingItem = state.items.find((item) => item.id === id);
            state.totalQuantity--;
            state.changed = true;
            if (existingItem.quantity ===1) {
                state.items = state.items.filter(item => item.id !==id);
            } else {
                existingItem.quantity--;
                existingItem.totalPrice = existingItem.totalPrice - existingItem.price
            }
        }
    }
})
 
 
 
export const cartActions = cartSlice.actions
 
export default cartSlice
```

```javascript
import Cart from './components/Cart/Cart';
import Layout from './components/Layout/Layout';
import Products from './components/Shop/Products';
import {useSelector, useDispatch} from 'react-redux'
import {useEffect, Fragment} from 'react'
import Notification from './components/UI/Notification';
import {sendCartData, fetchCartData} from './store/cart-actions'
 
let isInitial = true
 
function App() {
  const dispatch = useDispatch();
  const showCart = useSelector(state => state.ui.cartIsVisible)
  const cart = useSelector((state) => state.cart);
  const notification = useSelector(state => state.ui.notification)
 
  useEffect(()=>{
    dispatch(fetchCartData())
  },[dispatch])
 
  useEffect(()=>{
 
    if (isInitial) {
      isInitial = false;
      return;
    }
if(cart.changed) {
  dispatch(sendCartData(cart))
}
  },[cart, dispatch])
 
  return (
    <Fragment>
    {notification &&
    (<Notification
      status={notification.status}
      title={notification.title}
      message={notification.message}
    />)}
    <Layout>
      {showCart && <Cart />}
      <Products />
    </Layout>
    </Fragment>
  );
}
 
export default App;
```

```javascript
import {uiActions} from './ui-slice'
import {cartActions} from './cart-slice'
export const fetchCartData = (cart) => {
    return async (dispatch) => {
        const fetchData = async () => {
            const response = await fetch(
                'https://react-http-26861-default-rtdb.firebaseio.com/cart.json'
            );
 
            if (!response.ok) {
                throw new Error('Could not fetch cart data')
            }
            const data = await response.json()
 
            return data;
        }
 
        try {
           const cartData = await fetchData();
           dispatch(cartActions.replaceCart(
            {
                items: cartData.items || [],
                totalQuantity: cartData.totalQuantity
            }
           ));
        } catch(error) {
            dispatch(
                uiActions.showNotification({
                  status: 'error',
                  title: 'Error!',
                  message: 'Fetching cart data failed',
                })
              )
        }
    }
}
 
export const sendCartData = (cart) => {
    return async (dispatch) => {
        dispatch(
            uiActions.showNotification({
              status: 'pending',
              title: 'Sending...',
              message: 'Sending cart data',
            })
          )
 
          const sendRequest = async () => {
            const response = await fetch(
                'https://react-http-26861-default-rtdb.firebaseio.com/cart.json',
                {
                  method:'PUT',
                  body: JSON.stringify({
                    items: cart.items,
                    totalQuantity: cart.totalQuantity}),
                })
         
                if (!response.ok) {
                  throw new Error('Sending cart data failed');
                }
          }
 
          try {
            await sendRequest();
 
            dispatch(
                uiActions.showNotification({
                  status: 'success',
                  title: 'Success!',
                  message: 'Sent cart data successfully',
                })
              )
          } catch (error) {
            dispatch(
                uiActions.showNotification({
                  status: 'error',
                  title: 'Error!',
                  message: 'Sent cart data failed',
                })
              )
          }
         
    }
}

```

