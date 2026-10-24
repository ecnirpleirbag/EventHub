# EventHub API

A backend REST API for a simplified event ticketing platform built with **Django** and **Django REST Framework**.

---

## How to Run the Project

```bash
# 1. (Optional) create and activate a virtual environment
python -m venv venv
venv\Scripts\activate      # Windows
# source venv/bin/activate  # macOS / Linux

# 2. Install dependencies
pip install django djangorestframework

# 3. Apply migrations
python manage.py makemigrations
python manage.py migrate

# 4. Start the development server
python manage.py runserver
```

The API is available at `http://127.0.0.1:8000/api/`.

---

## Endpoints

### Events

| Method | URL | Description |
|--------|-----|-------------|
| GET | `/api/events/` | List all events (supports `?status=` and `?venue=` filters) |
| POST | `/api/events/` | Create a new event |
| GET | `/api/events/{id}/` | Retrieve a single event |
| PUT/PATCH | `/api/events/{id}/` | Update an event |
| DELETE | `/api/events/{id}/` | Delete an event |

### Reservations

| Method | URL | Description |
|--------|-----|-------------|
| GET | `/api/reservations/` | List all reservations (supports `?event_id=` filter) |
| POST | `/api/reservations/` | Create a reservation (deducts seats from the event) |
| GET | `/api/reservations/{id}/` | Retrieve a single reservation |
| PUT/PATCH | `/api/reservations/{id}/` | Update a reservation |
| DELETE | `/api/reservations/{id}/` | Delete a reservation |
| POST | `/api/reservations/{id}/cancel/` | Cancel a reservation (restores seats to the event) |

---

## Sample Requests

**Create Event**
```json
POST /api/events/
{
  "title": "PyCon India 2025",
  "venue": "NIMHANS Convention Centre, Bangalore",
  "date": "2025-09-20",
  "total_seats": 500,
  "available_seats": 500,
  "status": "upcoming"
}
```

**Reserve Seats**
```json
POST /api/reservations/
{
  "event": 1,
  "attendee_name": "Priya Sharma",
  "attendee_email": "priya@example.com",
  "seats_reserved": 2
}
```

**Cancel Reservation**
```
POST /api/reservations/1/cancel/
(no request body required)
```

---

## Design Decision

**Seat deduction inside the serializer's `create()` method**

When a reservation is created, `available_seats` on the related `Event` is decremented inside `ReservationSerializer.create()`. This keeps both writes (update the event, insert the reservation) in the same method so it is easy to find and reason about. The validation that runs before `create()` ensures we never over-deduct seats. In a production system with high concurrency, this would be wrapped in `transaction.atomic()` combined with a `select_for_update()` query to prevent race conditions — that is left for a future iteration.

---

## Project Structure

```
eventhub/
├── manage.py
├── requirements.txt
├── README.md
├── eventhub/
│   ├── settings.py
│   └── urls.py
└── events/
    ├── models.py
    ├── serializers.py
    ├── views.py
    ├── urls.py
    ├── middleware.py
    └── migrations/
```
